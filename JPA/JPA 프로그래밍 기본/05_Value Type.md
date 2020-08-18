<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [Embedded, Embeddable](#embedded-embeddable)
- [불변 객체](#불변-객체)
- [값 타입 컬렉션](#값-타입-컬렉션)


### Embedded, Embeddable
- 모듈 형식으로 객체를 조립하여 하나의 테이블로 사용이 가능하다.
- 다른 객체를 embedded하여  객체지향스러운 설계가 가능하고 객체와 테이블을 아주 세밀하게 (find-grained) 매핑하는 것이 가능하다.
- 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많다.
- Embedded 객체 내부에 다른 Entity 포함이 가능하다.
- Embedded 되어있는 객채가 null 이면 내부의 연관된 모든 필드가 null값이 된다.

하나의 객체에서 두개의 같은 객체를 Embedded 할때는 컬럼명 중복으로 인한 에러가 발생하게 되는데 이때 사용되는게 `@AttributeOverrides`, `@AttributeOverride` 어노테이션이다.

``` kotlin
@Entity
class Member : BaseEntity() {
    @get:Id
    @get:GeneratedValue
    var id: Long? = null
    var name: String? = null
    var age: Int? = null

    @get:ManyToOne(fetch = FetchType.LAZY)
    @get:JoinColumn(name = "teamId")
    var team: Team? = null

    @Embedded
    var period: Period? = null

    @Embedded
    var homeAddress: Address? = null

    @get:Embedded
    @get:AttributeOverrides(
        AttributeOverride(
            name = "city",
            column = Column(name = "WORK_CITY")
        ),
        AttributeOverride(
            name = "street",
            column = Column(name = "WORK_STREET")
        ),
        AttributeOverride(
            name = "zipcode",
            column = Column(name = "WORK_ZIP_CODE")
        )
    )
    var workAddress: Address? = null
} 

@Embeddable
class Period {
    var startDate: LocalDateTime = LocalDateTime.now()
    var endDate: LocalDateTime = LocalDateTime.now()
}

@Embeddable
class Address {
    var city: String? = null
    var street: String? = null
    var zipcode: String? = null
}
```

### 불변 객체
임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험함
하나의 값만 수정해도 공유된만큼의 여러번의 업데이트가 나가서 사이드 이펙트가 발생할 우려가 크다.
공유를 의도하고 설계가 되었다면 Embedded 방식이 아닌 Entity 방식으로 해야한다.
같은 값이 들어가는걸 만들고 싶다면 소프트 카피가 아닌 딥카피를 하여 사용하여야한다.

그래서 이를 방지하기 위해선 값이 변하지 않게 불변객체를 만들어서 사용하여야한다 (자바의 경우 setter를 제거한다)
만약 값의 수정이 필요하여 새로 변경해야할 경우에는 객체를 새로 생성하여 교체하는 방식으로 하여야한다.

> A라는 객체와 B라는 객체가 Z1라는 Embedded 객체를 공유하고있다고 가정할경우
> A, B 둘중에 하나만 수정하여도 업데이트가 두번 발생하여 A, B 둘다 변경된 Z1 값을된다.
> 그리하여 값을 수정할때는 새로운 객체 Z2를 만들어 Z1의 필드 값을 복사한뒤 변경해줘야한다.

### 값 타입 컬렉션

- 데이터베이스는 컬렉션을 같은 테이블에 저장 할 수 없어 컬렉션을 저장하기 위해 별도의 테이블이 필요하다
- 값 타입을 하나 이상 저장할 때 `@ElementCollection`, `@CollectionTable` 사용하여 엔티티를 사용하지 않아도 별도의 테이블 생성이 가능하다.

``` kotlin
@Entity
class Member : BaseEntity() {
    @get:Id
    @get:GeneratedValue
    var id: Long? = null

    @get:ElementCollection
    @get:CollectionTable(
        name = "favoriteFood",
        joinColumns = [
            JoinColumn(name = "memberId")
        ]
    )
    var favoriteFoods: Set<String> = hashSetOf()

    @get:ElementCollection
    @get:CollectionTable(
        name = "address",
        joinColumns = [
            JoinColumn(name = "memberId")
        ] 
    )
    var addressHistory: List<Address> = mutableListOf()

}
```

> 값 타입 컬렉션은 영속성 전이(Cascade) + 고아객체 제거 기능을 필수로 가지고 있다고 볼 수 있다.
> 값 타입의 주체인 엔티티가 삭제되거나 그 내부에서 수정이 일어나면 자동으로 업데이트, 삭제 처리가 되어 모든 생명주기를 주체가 가지게 된다.
> 해당 엔티티를 디비에서 조회시 하위 컬렉션들은 자동적으로 지연로딩 기능을 가지게된다.


Collection 사용시 주의사항
> favoriteFoods 에서 하나의 항목 수정이 일어날 경우에는
> ``` kotlin
> member.favoriteFoods.remove("치킨")
> member.favoriteFoods.add("짜장면")
> ```
> 형태로 삭제한뒤에 추가시켜주는 방식으로 하여야 한다.

값 타입 컬렉션의 제약사항
- 값 타입은 엔티티와 다르게 식별자 개념이 없어 값이 변경되면 추적이 어렵다.
- 값 타입 컬렉션에 변경 사항이 발생하면 주인 엔티티와 연관된 모든 데이터를 삭제하고, 값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다.
- 값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성하여야한다. (null 입력 X, 중복 저장 X)
> **값 타입 컬렉션에 대안 **
> 실무에서는 상황에 라 값 타입 컬렉션 대신에 일대다 관계를 고려하는것이 좋다
> 영속성전이 + 고아객체 제거를 사용하여 값 타입 컬렉션처럼 사용
> (값타입 컬렉션으로 가저가기에 delete, insert되는 항목들이 너무 많으면 디비에 부하가 크게 걸릴 수 있으며 사이드 이펙트로 위험한 상황이 발생 할 수 있다.)
> 엔티티로 가져감으로 컨트럴 하기가 유연해지고 쿼리를 다양하게 적용할 수 있다.

값 타입 컬렉션은 셀렉트 박스 멀티 셀렉트에 사용하기 좋다.
식별자가 필요하고, 지속해서 값을 추적, 변경해야 한다면 그것은 값 타입이 아닌 엔티티로 설계해야한다.
