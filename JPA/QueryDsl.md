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

``` kotlin
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


