예전에는 OS에 설치하여 사용해야하는 많은것들이 이제는 `Docker`를 이용하여 쉽게 실행하고 종료시킬 수 있습니다.
Local 환경에서 테스트 해야하는 Database도 그중에 하나인데 보통 서비스 형태로 백그라운드에 항시 실행되어 있습니다.
그럴경우에는 리소스 낭비도 심하고 백그라운드 서비스들로 인하여 작업하는데 메모리가 적어 버벅이거나 하는 현상이 생깁니다.

그래서 도커를 이용하여 Database를 올렸다가 테스트가 끝나면 내리고 하는데 이것도 너무 번거롭고 자칫하면 계속 백그라운드에 남아있어 리소스를 잡아먹습니다.

그래서 나온게 [TestContainers](https://www.testcontainers.org/)입니다.  
TestContainers는 테스트 이전에 H2, PostgreSQL등 Docker Container를 따로 띄우지 않아도 자동으로 테스트할때 DB Contatiner를 자동으로 띄워주는 역할을 하는 라이브러리이다.

> 해당 예제는 Kotlin, KotlinDSL(gradle)로 작성되어있습니다.

## Gradle 설정법
``` kotlin
extra["testcontainersVersion"] = "1.14.3"

dependencies {
    /* testcontainers */
    testCompile("org.testcontainers:postgresql:1.14.3")
    testImplementation("org.testcontainers:junit-jupiter")
    
    /* 기타 예제에 필요한 의존성 */    
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    runtimeOnly("com.h2database:h2")
    testImplementation("org.springframework.boot:spring-boot-starter-test") {
        exclude(group = "org.junit.vintage", module = "junit-vintage-engine")
    }

}

dependencyManagement {
    imports {
        mavenBom("org.testcontainers:testcontainers-bom:${property("testcontainersVersion")}")
    }
}
```
위에는 postgresql을 사용하는 예제로 `org.testcontainers:postgresql`를 추가하였습니다.
Postgresql이 아닌 다른 데이터 베이스를 설정할때는 다음 링크에서 참조하여 알맞게 넣어주시면 됩니다.
- [PostgreSQL Module](https://www.testcontainers.org/modules/databases/postgres/)

> 모듈리스트에 없는 데이터베이스를 설정하는 방법에 대해서는 하단에 추가적으로 작성해두었습니다.

## Test Properties
`application-test.properties`
``` properties
spring.datasource.url=jdbc:tc:postgresql:///test-database
spring.datasource.driver-class-name=org.testcontainers.jdbc.ContainerDatabaseDriver
spring.jpa.hibernate.ddl-auto=create-drop
```
기존에 존재하던 Hostname과 Port 정보는 사용하지 않을 정보라 기입하지 않습니다.
데이터베이스는 테스트시에만 잠깐 존재하고 사라지는 형태라 Username, Password 정보도 필요가 없어 명시하지 않습니다.
`tc`라는 키워드를 가진 URL을 JDBC 드라이버가 사용하도록 `spring.datasource.driver-class-name`에 `ContainerDatabaseDriver`를 설정하여 줍니다.

## Code

간단한 예제로 회원정보를 담는 `Member`와 회원정보를 저장할 `MemberRepository`를 생성합니다.
``` kotlin
// Member.kt
@Entity
class Member(
    @Id @GeneratedValue
    var id: Long? = null,
    var name: String? = null,
    var email: String? = null,
    var password: String? = null
)

// MemberRepository.kt
interface MemberRepository : JpaRepository<Member, Long>
```

``` kotlin
// MemberRepositoryTest.kt
package com.ssabae.container.test

import com.ssabae.container.test.domain.Member
import com.ssabae.container.test.domain.MemberRepository
import org.assertj.core.api.Assertions.assertThat
import org.junit.jupiter.api.AfterAll
import org.junit.jupiter.api.BeforeAll
import org.junit.jupiter.api.Test
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest
import org.testcontainers.containers.PostgreSQLContainer
import org.testcontainers.junit.jupiter.Testcontainers

@DataJpaTest
class MemberRepositoryTest {

    companion object {

        @JvmStatic
        private val postgresqlContainer = PostgreSQLContainer<Nothing>("postgres:latest").apply {
            withDatabaseName("test-database")
        }

        @BeforeAll
        @JvmStatic
        fun beforeAll() {
            postgresqlContainer.start()
        }

        @AfterAll
        @JvmStatic
        fun afterAll() {
            postgresqlContainer.stop()
        }
    }

    @Autowired
    lateinit var memberRepository: MemberRepository

    @Test
    fun test() {
        val member = Member(
            email = "email",
            name = "name",
            password = "password"
        )
        val save = memberRepository.save(member)
        assertThat(save.email).isEqualTo("email")
        assertThat(save.name).isEqualTo("name")
        assertThat(save.password).isEqualTo("password")
    }
}
```
`PostgreSQLContainer`에서 설정한 DatabaseName은 properties에서 명시한 DatabaseName과 일치해야합니다.

`PostgreSQLContainer`가 static이 아니라면 각각 모든 테스트마다 컨테이너가 새로 생성되어 속도가 매우 느려지게 됩니다.

`@BeforeEach`를 이용하여 매 테스트마다 내부에 있는 `Entity`들을 삭제하여줍니다.


다음과 같이 JUnit5에서 제공하는 beforeAll, afterAll에서 Contatiner를 시작하는 로직을 삭제할 수 있습니다.
``` kotlin
@DataJpaTest
@Testcontainers
class MemberRepositoryTest {
    
    companion object {
        @JvmStatic
        @Container
        private val postgresqlContainer = PostgreSQLContainer<Nothing>("postgres:latest").apply {
            withDatabaseName("test-database")
        }
    }
    
    @Autowired
    lateinit var memberRepository: MemberRepository

    @BeforeEach
    fun beforeEach() {
        memberRepository.removeAll()
    }
}
```
`@Testcontainers` + `@Container` 두개의 Annotation으로 생명주기를 자동으로 관리 할 수 있습니다.

## 미지원 모듈 설정 방법
``` kotlin
@DataJpaTest
@Testcontainers
class MemberRepositoryTest {

    companion object {
        @JvmStatic
        @Container
        private val h2Container = GenericContainer<Nothing>("oscarfonts/h2").apply {
            withExposedPorts(1521)
        }
    }
}
```

`GenericContainer`는 위에서 봤던 `PostgreSQLContainer`의 슈퍼클래스입니다.
생성자의 파라미터로 로컬에 설치되어있는 이미지를 먼저 검색하고 없으면 Docker Hub에서 다운로드 받아서 실행해줍니다.
GenericContainer로 Container를 생성해주게 된다면 별도의 properties가 없어도 됩니다.


## Contatiner Logging
``` kotlin
class MemberRepositoryTest {

    companion object {

        private val LOGGER = LoggerFactory.getLogger(MemberRepositoryTest::class.java)

        @JvmStatic
        @Container
        private val h2Container = GenericContainer<Nothing>("oscarfonts/h2").apply {
            withExposedPorts(1521)
        }

        @BeforeAll
        @JvmStatic
        fun beforeAll() {
            val logConsumer = Slf4jLogConsumer(LOGGER)
            h2Container.followOutput(logConsumer)
        }
    }
    
    @Test
    fun test() {
        val member = Member(
            email = "email",
            name = "name",
            password = "password"
        )
        val save = memberRepository.save(member)
        assertThat(save.email).isEqualTo("email")
        assertThat(save.name).isEqualTo("name")
        assertThat(save.password).isEqualTo("password")

        LOGGER.debug("logs : ${h2Container.logs}")
    }

}
```
Container에서 제공해주는 followOutput 함수에 Logger를 넣거나 Lambda식을 사용하여 아웃풋 데이터를 로그를 남길 수 있습니다.
또 `getLog()` 를 이용하여 쌓여있던 로그를 받아올 수 있습니다.

## Docker Compose에서의 사용

[Docker Compose](https://docs.docker.com/compose/)에서 사용하는 방법에 대해서는 `docker-compose.yml` 파일을 생성해주면서 시작이 됩니다.

``` yaml
// docker-compose.yml
version: "3"

services:
  study-db:
    image: postgres
    ports:
      - 5432
    environment:
      POSTGRES_DB: database_name
      POSTGRES_USER: database_user
      POSTGRES_PASSWORD: database_password
```
`docker-compose.yml` 파일은 `src/test/resources/` 아래에 있게 하는게 가장 관리하게 편하고 좋은 위치인거 같습니다.
``` kotlin
// MemberRepositoryTest.kt
@DataJpaTest
@Testcontainers
class MemberRepositoryTest {

    companion object {

        @JvmStatic
        @Container
        private val container = 
            DockerComposeContainer<Nothing>(
            	File("src/test/resources/docker-compose.yml")
            )
    }
    ...
}
```
위와같이 DockerComposeContainer에서 File로 yml파일을 불러 넣어주면 자동적으로 docker-compose가 적용이 되면서 contatiner가 뜨게 됩니다.


## 단점?
단점이라고하면 도커 컨테이너를 준비하고 실행하는 시간이 좀 오래 걸립니다.
로컬에서만 해야하는 테스트를 빠르고 간편하게 하기위해서만 사용하는것을 추천드립니다.

하지만 설정하는법도 간단하고 적용해보는데 시간도 얼마 안들어가 경험삼아 도입을 해보시는것도 나쁘지는 않은거라 생각됩니다.
