# QueryDsl
복잡한 쿼리를 Java 언어로 제공하여줌으로 컴파일 시점에 문법 오류를 찾을 수 있고 복잡한 쿼리를 쉽게 작성할 수 있다.

## 설정
``` kotlin
// build.gradle.kt
plugins {
    id("com.ewerk.gradle.plugins.querydsl") version "1.0.10"
    kotlin("kapt") version "1.3.72"
    idea
}

buildscript {
    repositories {
        maven("https://plugins.gradle.org/m2/")
        mavenCentral()
    }
    dependencies {
        classpath("gradle.plugin.com.ewerk.gradle.plugins:querydsl-plugin:1.0.10")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:1.3.61")
    }
}

dependencies {
    implementation("com.querydsl:querydsl-jpa")
    kapt("com.querydsl:querydsl-apt:4.2.2:jpa")
    kapt("org.hibernate.javax.persistence:hibernate-jpa-2.1-api:1.0.2.Final")
}

idea {
    module {
        val kaptMain = file("build/generated/source/kapt/main")
        sourceDirs.add(kaptMain)
        generatedSourceDirs.add(kaptMain)
    }
}
```
- `com.querydsl:querydsl-apt` : Code Generation Library
- `com.querydsl:querydsl-jpa` : 실제 JPA 쿼리를 간편하게 사용하는 Library


여기서 중요한 점 은 annotation processing을 할 수 있도록 `kapt`를 사용하는 것과, `querydsl-apt` `dependency` 마지막에 `:jpa`를 붙여주는 것이다.

`gradle compileKotlin` 명령어를 이용하여 Q 파일을 생성한다.

``` yaml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/querydsl
    username: sa
    password:
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
#        show_sql: true # 밑에 org.hibernate.SQL: debug 옵션과 같은 역할을 한다.
        format_sql: true  # 쿼리를 보여준다.
        use_sql_comments: true # JPQL 쿼리를 보여준다.

logging.level:
  org.hibernate.SQL: debug
  org.hibernate.type: trace
```



## 기본 사용법
``` kotlin
val jpaQueryFactory = JPAQueryFactory(em)
val m = QMember.member
val member = jpaQueryFactory
    .select(m)
    .from(m)
    .where(m.name.eq("member1"))
    .fetchOne()
```
Querydsl로 작성을한 코드는 JPQL 형태로 변환이 되어 쿼리가 나가게 된다.


### QType 활용
``` kotlin
// 별칭을 이용한 사용
val m = Qmember("m")

// static 인스턴스 직접 사용
val m = QMember.member
```
QType 객체 생성시 alias를 따로 주어 같은 Table 내에서 join이 발생시 구분하여 사용이 가능하지만
보통 그런 경우는 많이 없기 때문에 static instance를 직접 사용하는 방식이 많이 쓰인다.

### 검색 조건

``` java
// username = 'member1'
member.username.eq("member1") 
member.username.ne("member1") //username != 'member1'
member.username.eq("member1").not() // username != 'member1'

member.username.isNotNull() //이름이 is not null

member.age.in(10, 20) // age in (10,20)
member.age.notIn(10, 20) // age not in (10, 20)
member.age.between(10,30) //between 10, 30

member.age.goe(30) // age >= 30
member.age.gt(30) // age > 30
member.age.loe(30) // age <= 30
member.age.lt(30) // age < 30 

member.username.like("member%") //like 검색
member.username.contains("member") // like ‘%member%’ 검색
member.username.startsWith("member") //like ‘member%’ 검색
```

### Fetch 종류

|fetch Type|설명|
|:--|:--|
|fetch()|리스트 조회, 데이터 없으면 빈 리스트 반환|
|fetchOne()|한건만 조회<br>결과가 없으면 null을 리턴<br>결과가 둘 이상이면 `com.querydsl.core.NonUniqueResultException`|
|fetchFirst()|limit(1).fetchOne()|
|fetchResults()|QueryResult Type을 반환하여 페이징 정보 포함, total count 정보를 같이 반환<br>results.getTotal() results.getResults() 를 통하여 값이나 정보를 획득|
|fetchCount()|count 쿼리로 변경해서 count 수 조회|

`fetchResults` 같은 경우는 복잡한 쿼리에 대해서는 Count 쿼리랑 Select 쿼리랑 서로 조건이맞지 않아 다른 결과를 낼 수 있음으로 복잡한 쿼리에 대해서는 사용하지 않는것을 권장


### Sorting
``` kotlin
    val members: List<Member> = queryFactory
        .select(member)
        .from(member)
        .where(member.age.eq(100))
        .orderBy(member.age.asc(), member.name.asc().nullsLast())
        .fetch()
```

### Paging
``` kotlin
    val members = queryFactory.selectFrom(member)
        .offset(1)
        .limit(2)
        .fetch()
```

### Aggregation
``` kotlin
    val result: List<Tuple> = queryFactory
        .select(
            member.age.count(),
            member.age.sum(),
            member.age.avg(),
            member.age.max( ),
            member.age.min()
        )
        .from(member)
        .fetch()

    val tuple = result[0]
    assertThat(tuple.get(member.age.count())).isEqualTo(4)
    assertThat(tuple.get(member.age.sum())).isEqualTo(100)
    assertThat(tuple.get(member.age.avg())).isEqualTo(25.0)
    assertThat(tuple.get(member.age.max())).isEqualTo(40)
    assertThat(tuple.get(member.age.min())).isEqualTo(10)
```

|function|설명|
|:--|:--|
|COUNT|row count|
|SUM|합|
|AVG|평균|
|MAX|최대값|
|MIN|최소값|

### Grouping
``` kotlin
    val result: List<Tuple> = queryFactory
        .select(team.name, member.age.avg())
        .from(member)
        .join(member.team, team)
        .groupBy(team.name)
        .having(member.age.avg().gt(10))
        .fetch()

    val teamA = result[0]
    val teamB = result[1]

    assertThat(teamA.get(team.name)).isEqualTo("A")
    assertThat(teamA.get(member.age.avg())).isEqualTo(15.0)

    assertThat(teamB.get(team.name)).isEqualTo("B")
    assertThat(teamB.get(member.age.avg())).isEqualTo(35.0)
```

## Join

### Normal Join

조인의 기본 문법으로는 `join(first, second)` first에 조인 대상을 지정하여준 뒤에 second에 조인할 파라메터의 별팅으로 사용할 QType을 지정하여주면 된다.

``` kotlin
@Test
fun join() {
    val members = queryFactory
        .selectFrom(member)
        .join(member.team, team)
        .where(team.name.eq("A"))
        .fetch()

    assertThat(members).extracting("name")
        .containsExactly("member1", "member2")
}
```
### Theta Join
연관관계가 맺어져있지 않은 전혀 관련이 없는 두 조건을 Join하여준다.


``` kotlin
@Test
fun theta_join() {
    em.persist(Member(name = "A"))
    em.persist(Member(name = "B"))
    em.persist(Member(name = "C"))

    val members = queryFactory
        .select(member)
        .from(member, team)
        .where(member.name.eq(team.name))
        .fetch()

    assertThat(members).extracting("name")
        .containsExactly("A", "B")
}
```
Theta Join은 From절에 여러 Entity를 선택해서 조인이 가능
하지만 외부 조인이 불가능하고 inner join으로만 가능하다.
하지만 on을 이용할 경우에는 외부 조인이 가능하다.

### Using On
JPA 2.1부터 제공되는 옵션으로 On 절을 활용한 Join 방법에는 크게 두가지 활용법이 있다.
1. 조인 대상 필터링
2. 연관관계 없는 엔티티의 외부조인

#### 내부 Join
``` kotlin
    val members1 = queryFactory
        .select(member, team)
        .from(member)
        .join(member.team, team)
            .on(team.name.eq("A"))
        .fetch()
    
    val members2 = queryFactory
        .select(member, team)
        .from(member)
        .join(member.team, team)
        .where(team.name.eq("A"))
        .fetch()
```
> on 절을 활용해 조인 대상을 필터링 할 때, 외부조인(left)이 아니라 내부조인(inner join)을 사용하면,
> where 절에서 필터링 하는 것과 기능이 동일하다. 따라서 on 절을 활용한 조인 대상 필터링을 사용할 때,
> 내부조인 이면 익숙한 where 절로 해결하고, 정말 외부조인이 필요한 경우에만 이 기능을 사용하자.

#### 외부 Join
``` kotlin
    val members = queryFactory
        .select(member, team)
        .from(member)
        .leftJoin(member.team, team)
            .on(team.name.eq("A"))
        .fetch()
```
내부 join과는 다르게 where에서 on과 같은 상황을 주어도 결과가 같지 않아 leftJoin을 사용할 경우에는 on에서 해결해야한다.

#### Theta Join
``` kotlin
@Test
fun join_on_no_relation() {
    em.persist(Member(name = "A"))
    em.persist(Member(name = "B"))
    em.persist(Member(name = "C"))

    val members = queryFactory
        .select(member, team)
        .from(member)
        .leftJoin(team) // 
            .on(member.name.eq(team.name))
        .fetch()

    members.forEach { println("it = ${it}") }
}
```

하이버네이트 5.1부터 on 을 사용해서 서로 관계가 없는 필드로 외부 조인하는 기능이 추가 (내부 조인도 가능)
`leftJoin(team)` 대신 `leftJoin(member.team, team)`형태로 들어가게되면 ID 끼리 매칭이 되어버려 theta join 형태로 되지 않는다.
- 일반조인: leftJoin(member.team, team)
- on조인: from(member).leftJoin(team).on(xxx)

#### Fetch Join

페치 조인은 SQL에서 제공하는것이 아니고 JPQL에서 제공하는 기능이다.
SQL조인을 활용해서 연관된 엔티티를 SQL 한번에 조회하는 기능
주로 성능 최적화에 사용하는 방법이다

``` kotlin
@Test
fun fetchJoin() {

    val selectedMember = queryFactory
        .selectFrom(member)
        .join(member.team, team).fetchJoin()
        .where(member.name.eq("member1"))
        .fetchOne()

    val loaded = emf.persistenceUnitUtil.isLoaded(selectedMember?.team)

    assertThat(loaded).isTrue()
}

```
join을 설정하여 준 뒤에 fetchJoin()을 호출하기만 해주면 JPQL의 fetchJoin이 설정된다.



## SubQuery
`com.querydsl.jpa.JPAExpressions`을 사용하여 서브쿼리를 생성한다.

### Where Subquery
``` kotlin
@Test
fun subQueryInCase() {
    val subMember = QMember("subMember")
    val members = queryFactory
        .selectFrom(member)
        .where(
            member.age.`in`(
                JPAExpressions
                    .select(subMember.age)
                    .from(subMember)
                    .where(subMember.age.gt(10))
            )
        )
        .fetch()

    assertThat(members).extracting("age")
        .containsExactly(20, 30, 40)
}
```

### Select Subquery
``` kotlin
@Test
fun selectSubQuery() {
    val subMember = QMember("subMember")
    val members = queryFactory
        .select(
            member.name,
            JPAExpressions
                .select(subMember.age.avg())
                .from(subMember)
        )
        .from(member)
        .fetch()


    members.forEach { println(it) }
}
```

### From Subquery

JPA JPQL 서브쿼리의 한계점으로 from 절의 서브쿼리(인라인 뷰)는 지원하지 않는다. 당연히 Querydsl도 지원하지 않는다.  
하이버네이트 구현체를 사용하면 select 절의 서브쿼리는 지원한다. Querydsl도 하이버네이트 구현체를 사용하면 select 절의 서브쿼리를 지원한다.

> from 절의 서브쿼리 해결방안
> 1. 서브쿼리를 join으로 변경한다. (가능한 상황도 있고, 불가능한 상황도 있다.)
> 2. 애플리케이션에서 쿼리를 2번 분리해서 실행한다.
> 3. nativeSQL을 사용한다.


## Case
간단한 방식의 CASE
``` kotlin
@Test
fun simpleCase() {
    val members = queryFactory
        .select(
            member.age
                .`when`(10).then("열살")
                .`when`(20).then("스무살")
                .otherwise("기타")
        )
        .from(member)
        .fetch()

    members.forEach { println(it) }
}

```


CaseBuilder를 사용한 Case
``` kotlin
@Test
fun complexCase() {
    val members = queryFactory
        .select(CaseBuilder()
            .`when`(member.age.between(0, 20)).then("0~20")
            .`when`(member.age.between(21, 30)).then("21~30")
            .otherwise("기타")
        )
        .from(member)
        .fetch()

    members.forEach { println(it) }
}
```


## 기타 기능

### Concat
문자열을 concat으로 합칠 수 있다.
age는 숫자 타입이라 stringValue를 이용하여 String 형태로 변경한다.
stringValue은 Enum을 처리할 때 자주 사용된다.
``` kotlin
@Test
fun concat() {
    val members = queryFactory
        .select(member.name.concat("_").concat(member.age.stringValue()))
        .from(member)
        .fetch()
}
```

### Distinct
``` kotlin
@Test
fun distinct() {
    val members = queryFactory
        .select(member).distinct()
        .from(member)
        .fetch()
}
```

### Function Call
``` kotlin
@Test
fun functionCall() {
    val members = queryFactory
        .select(
            Expressions.stringTemplate(
                "function('replace', {0}, {1}, {2})",
                member.name,
                "member",
                "M"
            )
        )
        .from(member)
        .fetch()

    members.forEach { println(it) }
    assertThat(members).containsExactly("M1", "M2", "M3", "M4")
}


@Test
fun selectLowerMember() {
    val members = queryFactory
        .selectFrom(member)
        // 대략적인 ANCI 표준 함수들은 querydsl에 내장되어있음
        .where(member.name.eq(member.name.lower()))
        .fetch()
}
```
## Projection

### Bean
Setter를 이용한 프로젝션
DTO에 setter가 생성될 수 있도록 val -> var 로 수정을 해주어야한다.
``` kotlin
// MemberDto.kt
class MemberDto(
    var name: String? = null,
    var age: Int? = null
) {
    override fun toString(): String {
        return "MemberDto(name=$name, age=$age)"
    }
}

// QueryDslTest.kt
@Test
fun projectionDto_Setter() {
    val members = queryFactory
        .select(
            Projections.bean(
                MemberDto::class.java,
                member.name,
                member.age
            )
        )
        .from(member)
        .fetch()

    members.forEach { println(it) }
}
```

### Field
Field에 직접 값을 저장하는 방식 (리플렉션으로 private field도 처리 가능)

``` kotlin
// MemberDto.kt
class MemberDto(
    val name: String? = null,
    val age: Int? = null
) {
    override fun toString(): String {
        return "MemberDto(name=$name, age=$age)"
    }
}

// QueryDslTest.kt
@Test
fun projectionDto_Field() {
    val members = queryFactory
        .select(
            Projections.fields(
                MemberDto::class.java,
                member.name,
                member.age
            )
        )
        .from(member)
        .fetch()

    members.forEach { println(it) }
}
```

만약 DTO에 있는 변수명이랑 Entity에 있는 field명이 다를시에는 alias를 이용하여 처리가 가능
``` kotlin
// MemberDto.kt
class MemberDto(
    val username: String? = null,
    val age: Int? = null
) {
    override fun toString(): String {
        return "MemberDto(username=$username, age=$age)"
    }
}

// QueryDslTest.kt
@Test
fun projectionDto_Field_alias() {
    val subMember = QMember("subMember")
    val members = queryFactory
        .select(
            Projections.fields(
                MemberDto::class.java,
                member.name.`as`("username"),
                ExpressionUtils.`as`(
                    JPAExpressions
                        .select(subMember.age.max())
                        .from(subMember)
                    ,"age"
                )
            )
        )
        .from(member)
        .fetch()

    members.forEach { println(it) }
}
```
`ExpressionUtils`을 이용하여 서브쿼리도 alias가 가능


### Constructor
생성자를 이용한 프로젝션
생성자에 입력하는 순서와 타입이 일치하여야한다.
``` kotlin
// MemberDto.kt
class MemberDto(
    val name: String? = null,
    val age: Int? = null
) {
    override fun toString(): String {
        return "MemberDto(name=$name, age=$age)"
    }
}

// QueryDslTest.kt
@Test
fun projectionDto() {
    val members = queryFactory
        .select(
            Projections.constructor(
                MemberDto::class.java,
                member.name,
                member.age
            )
        )
        .from(member)
        .fetch()

    members.forEach { println(it) }
}
```

### QueryProjection

DTO 생성자에 `@QueryProjection`를 추가하여준 뒤에 gradle에서 QueryDsl 빌드를 하게되면  
DTO를 Q 파일로 제공하여준다.
그 Q 파일을 이용하여 DTO를 반환하는 프로젝션을 만들 수 있다.
``` kotlin
// MemberDto.kt
class MemberDto @QueryProjection constructor(
    val name: String? = null,
    val age: Int? = null
) {
    override fun toString(): String {
        return "MemberDto(name=$name, age=$age)"
    }
}

```
``` java
// QMemberDto.java
/**
 * com.ssabae.querydsl.demo.domain.dto.QMemberDto is a Querydsl Projection type for MemberDto
 */
@Generated("com.querydsl.codegen.ProjectionSerializer")
public class QMemberDto extends ConstructorExpression<MemberDto> {

    private static final long serialVersionUID = 938934842L;

    public QMemberDto(com.querydsl.core.types.Expression<String> name, com.querydsl.core.types.Expression<Integer> age) {
        super(MemberDto.class, new Class<?>[]{String.class, int.class}, name, age);
    }

}
```

``` kotlin
@Test
fun projectionDto_QDto() {
    val members = queryFactory
        .select(QMemberDto(member.name, member.age))
        .from(member)
        .fetch()

    members.forEach { println("it = $it") }

}
```

장점으로는 생성자 만드는 상황에서 컴파일 에러를 잡을 수 있다.
기존의 `Projections.constructor`를 사용하는 방식은 가변인자 형태라 더 많은 인자들을 넣어도 컴파일 에러가 발생하지 않고 런타임 에러가 발생하게 된다.


단점으로는 DTO에서 QueryDSL에 대한 의존성을 가지게 되어 아키텍쳐 적인 단점이 매우 늘어난다.


