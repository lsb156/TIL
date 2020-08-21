# Auditing
> Java8이 나오기 전까지 사용되었던 Date와 Calendar 클래스는 다음같은 문제점이 있었음
> - 불변 객체가 아니다.
>     - 멀티 스레드 환경에서 언제든 문제가 발생
> - Calendar는 월(Month) 값 설계까 잘못 되었음
>     - 10월을 나타내는 Calendar.OCTOBER의 숫자값이 '9'
>
> JodaTime이라는 오픈소스를 사용해서 문제점들을 피했었고, Java8에서 LocalDate, LocalDateTime을 통해 해결
> [JAVA의 날자와 시간 API](https://d2.naver.com/helloworld/645609)

LocalDate, LocalDateTime이 데이터베이스에 제대로 매핑되지 않는 이슈가 Hibernate 5.2.10	에서 해결됨
(스프링부트 2.x에서는 문제 없음)

``` java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseTimeEntity {
    
    @CreatedDate
    private LocalDateTime createdDate;
    
    @LastModifiedDate
    private LocalDateTime modifiedDate;
    
}
```
1. `@MappedSuperClass`
	- JPA Entity클래스들이 BaseTimeEntity를 상속할 경우 필드들도 컬럼으로 인식하도록 설정
2. `@EntityListeners(AuditingEntityListener.class)`
	- BaseTimeEntity 클래스에서 Auditing 기능을 포함시킴
3. `@CreatedDate`
	- Entity 가 생성되어 저장될 때 시간이 자동 저장됩니다.
4. `@LastModifiedDate`
	- 조회한 Entity의 값을 변경할 때 시간이 자동 저장됨.

어플리케이션 내에 모두 적용이 되도록 어플리케이션 클래스 상단에 `@EnableJpaAuditing`을 선언하여준다
