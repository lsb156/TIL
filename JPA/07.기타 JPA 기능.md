<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [기타 JPA 기능](#기타-jpa-기능)
  - [NamedQuery](#namedquery)
  - [벌크연산](#벌크연산)


## 기타 JPA 기능

### NamedQuery
``` kotlin
@Entity
@NamedQuery(
	name = "Member.findByUsername",
	query = "select m from Member m where m.username = :username")
class Member {
	...
}

val resultList: List<Member> = em.createQuery("Member.findByUsername", Member::class.java)
	.setParameter("username", "회원1")
	.getResultList()
```

- 미리 정의하여 이름을 부여해두고 사용하는 JPQL
- 정적 쿼리만 가능 (동적쿼리가 가능하지 않다)
- 어노테이션, XML에 정의하여 사용한다.
- 애플리케이션 로딩 시점에 초기화 후 재사용 (애플리케이션 실행시점에 쿼리를 파싱하여 캐싱해놓음으로 퍼포먼스에 약간의 이점이 있음)
- **애플리케이션 로딩 시점에 쿼리를 검증** (미리 파싱하여 쿼리 오류 검증이 가능하다.)

> NamedQuery 이름은 관례상 {엔티티명}.{쿼리이름} 으로 한다.
> `Spring Data JPA`에서 `JpaRepository` 내부 함수에 직접 선언하는 쿼리가 네임드 쿼리로 등록이되어 실행시에 쿼리를 파싱하여 오류를 검증해준다.
> (익명 네임드쿼리라고 불리운다.)
> ``` kotlin
> interface UserRepository : JpaRepository<User, Long> {
>     @Query("select u from User u where u.emailAddress = ?1")
>     fun findByEmailAddress(emailAddress: String)
> }
> ```

### 벌크연산

시나리오 : 만약 재고가 10개 미만인 모든 상품의 가격을 10% 상승하려면?
- JPA 변경 감지 기능으로 실행하려면 너무 많은 SQL이 실행된다.
	1. 재고가 10개 미만인 상품을 리스트로 조회한다.
	2. 상품 엔티티의 가격을 10% 증가시킨다.
	3. 트랜잭션 커밋 시점에 변경감지가 동작한다.
- 변경된 데이터가 100건이라면 100번의 UPDATE SQL 실행

#### 벌크 연산 예제
- 쿼리 한 번으로 여러 테이블 로우 변경 (엔티티)
- executeUpdate()의 결과는 영향받은 엔티티 수 반환
- UPDATE, DELETE 지원
- INSERT (insert into .. select, 하이버네이트에서 지원, JPA 표준 스펙은 아님)
``` kotlin
val qlString = """
	update Product p
	set p.price = p.price * 1.1
	where p.stockAmount < :stockAmount
	"""
val resultCount = em.createQuery(qlString)
	.setParameter("stockAmount", 10)
	.executeUpdate()
```
Spring Data JPA에서는 `@Modifying` 을 지원하여 벌크 연산을 지원한다. 공식 문서에도 `clearAutomatically`
를 `true`로 주고 사용하여야한다.

#### 벌크 연산 주의점
벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리한다. 그러면서 영속성 컨텍스트와 디비의 값이 다른 문제점이 발생한다.

> 해결방법은 두가지가 있다.
> 1. 벌크 연산을 먼저 실행 (영속성 컨텍스트에 아무것도 남지 않도록 트렌잭션을 한번 끝낸다.)
> 2. 벌크연산 수행 후 영속성 컨텍스트 초기화