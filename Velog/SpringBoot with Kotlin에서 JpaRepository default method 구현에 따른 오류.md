## 오류의 발견
Spring Boot 환경에서 Kotlin 언어로 JpaRepository를 아래와 같은 코드로 구현하는 중이였습니다.

``` kotlin
// kotlin
interface AccountRepository : JpaRepository<Account, Long> {

    fun findByIdAndStateIn(id: Long, state: AccountState): Optional<Account>

    fun findByIdOnActive(id: Long): Optional<Account> {
        return this.findByIdAndStateIn(id, AccountState.ACTIVE)
    }
}
```
기대하고 있던 기능들은 다음과 같습니다.
1. `findByIdAndStateIn` 메소드는 parameter값으로 들어온 아이디와 계정 상태값에 대한 계정을 찾는다.
2. `findByIdOnActive` 메소드는 활성화된 계정 중 특정 아이디값을 찾는다

하지만 기대하는 결과와 다르게 실행을하면 다음과 같은 에러를 뱉어냅니다.
```
BeanCreationException: Error creating bean with name 'accountRepository': 
	FactoryBean threw exception on object creation; 
nested exception is java.lang.IllegalArgumentException: Failed to create query for method public abstract 
    java.util.Optional ...AccountRepository.findByIdOnActive
    (long)! No property onActive found for type Long! Traversed path:
    	Account.id.
```
처음에는 엔티티 설정 오류인줄 알았으나 `findByIdOnActive`, `onActive`라는 에러 로그의 단어를 검색하다가 찾아내게 되었습니다.

여기저기 한참 삽질을(몇시간...ㅠㅠ) 거듭하면서 알아냈단 오류의 키워드는 `No property onActive found`이며 오류의 원인은 다음과 같았습니다.


## 오류의 원인
Spring-Data에서는 `Query Creation`이라는 기능을 제공하는데 메소드명을 분석하여 알아서 쿼리를 만들어 주는 기능이 있는데 이것이 자동으로 동작해버렸습니다.
> [Query Creation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)
쿼리 빌더 메커니즘은 스프링 데이터 리파지토리 인프라스트럭쳐로 짜여져, 리파지토리의 엔티티들에 맞는 쿼리들을 만들어내는데 이 메커니즘은 find…By, read…By, query…By, count…By, 와 get…By같은 접두어들을 메소드에서 떼어내고 나머지부분을 파싱하여 쿼리를 만든다.


Java에서는 아래와 같이 default로 직접 인터페이스 내부에서 구현하여 자동으로 쿼리를 만들어주는것을 우회하고 있어 Kotlin도 당연히 인터페이스 내부에서 구현하면 default가 붙어서 생성될 것을 기대하고있었습니다.
``` java
// java
public interface AccountRepository extends JpaRepository<Account, Long> {

    Optional<Account> findByIdAndStateIn(Long id, AccountState state);
    
    default Optional<Account> findByIdOnActive(Long id) {
        return this.findByIdAndStateIn(id, AccountState.ACTIVE);
    }
}
```
> java8 이상부터 지원하는 기능중에 `Interface`에 `default` 키워드를 사용하여 메소드를 구현할 수 있는 기능이 지원되었다.

아까의 Kotlin 코드가 어떤 자바 코드로 변경이 되었는지 확인을 해보았습니다.
```java
public interface AccountRepository extends JpaRepository {
   @NotNull
   Optional findByIdAndStateIn(long var1, @NotNull Set var3);

   @NotNull
   Optional findByIdOnActive(long var1);

   @Metadata(
      mv = {1, 1, 16},
      bv = {1, 0, 3},
      k = 3
   )
   public static final class DefaultImpls {
      @NotNull
      public static Optional findByIdOnActive(AccountRepository $this, long id) {
         return $this.findByIdAndStateIn(id, SetsKt.setOf(new AccountState[]{AccountState.ACTIVE, AccountState.LOCKED}));
      }
   }
}
```
일반적으로 default 키워드가 붙질 않아 Spring Data에서 `findByIdOnActive` 메소드명을 쿼리로 변환하는데 `OnActive`라는 키워드를 파싱하지 못하여 에러가 났던거였습니다.

## 해결방법
### default 키워드를 붙여보자.
Kotlin에서는 [@JvmDefault](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/-jvm-default/)라는 어노테이션을 이용하여 default를 강제로 붙일 수 있습니다.
하지만 이렇게 하기위해서 실행할때 `-Xjvm-default=enable`라는 옵션을 주어 실행해주어야하고 아직은 실험중인 단계로보여 이 방법은 배제하기로 하였습니다.

### Companion object에 구현한다.
companion object에 별도로 구현하여 자동으로 쿼리를 만들지 못하게 만드는 방법도 있습니다.

하지만 companion 영역에 들어가면서 사용하는 방법을 변경해야 하는데 코드는 아래와 같습니다.

``` kotlin
interface AccountRepository : JpaRepository<Account, Long> {

    companion object {
        fun findByIdOnActive(accountRepository: AccountRepository, id: Long): Optional<Account> {
            return accountRepository.findByIdAndStateIn(id, AccountState.ACTIVE)
        }
    }
    
    fun findByIdAndStateIn(id: Long, states: AccountState): Optional<Account>
}

// 선언
@Autowired
lateinit var accountRepository: AccountRepository
// 호출
AccountRepository.findByIdOnActive(this.accountRepository, actual.id)
accountRepository.findByIdAndStateIn(actual.id, )
```
의도하는바와 결과가 같게 나와서 일단은 성공적이라고는 하지만 `findByIdOnActive`메소드를 호출하는 부분이 뭔가 깔끔하지 않습니다.. 그래서 이것도 후순위로 미뤄둡니다.

### Service Layer 구성
원래 설계당시부터 그랬어야했고 서비스 레이어의 목적과 용도를 생각해 보았을때 `findByIdExcludeDeleted`를 애초당시 서비스 레이어로 가져가는게 더 맞는 설계라고 생각합니다. 그래서 아래와 같이 서비스 레이어로 해당 메소드를 내려주었습니다.
``` kotlin
interface AccountRepository : JpaRepository<Account, Long> {
    fun findByIdAndStateIn(id: Long, states: Set<AccountState>): Optional<Account>
}


@Service
class AccountService(val accountRepository: AccountRepository) {
    fun findByIdAndStateIn(id: Long?, states: Set<AccountState>): Account? {
        if (id == null) return null
        return accountRepository.findByIdAndStateIn(id, states).orElse(null)
    }

    fun findByIdExcludeDeleted(id: Long): Account? {
        return this.findByIdAndStateIn(id, setOf(AccountState.ACTIVE, AccountState.LOCKED))
    }
}
```
이 구현이 다른 방법보다 훨씬 더 깔끔한 결과가 나왔다고 생각합니다.


## 결과
자동으로 쿼리를 만들어주는 옵션을 끄는 방법도 있을 거 같지만 해당 방법은 제가 아직 부족하여 찾지를 못하였습니다. 나중에 찾는대로 업데이트 하도록 하겠습니다. (혹시나 아시는분 계시면 댓글로 알려주시면 대단히 감사하겠습니다.)
하지만 옵션을 끈다고 하더라도 하나하나 쿼리를 다 구현해줘야 하는 방법은 오히려 미래적으로 더 악영향을 미칠거같다고 생각합니다.

하지만 해당 오류는 서비스 레이어를 안쓰고 곧바로 Repository에 테스트하면서 나왔던 오류로 정상적인 패턴대로 서비스 레이어를 구현했다면 나오지 않았을 오류라고 생각합니다.

아직은 해당 오류에 대한 피드와 블로그가 없기에 삽질한 후기 정리하여 공유합니다.