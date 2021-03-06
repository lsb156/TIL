# JUnit5

WebApplication에서  93%의 사용자가 테스트 유닛으로 JUnit을 사용한다.
(Mockito는 51%)

Spring Boot 2.2에서 `JUnit4` -> `JUnit5`로 업데이트


## JUnit 5 기초
![tdd-junit5-package](../asset/Test/tdd/tdd-junit5-package.png)

JUnit5는 크게 세 개의 요소로 구성되어 있다.
- **JUnit Platform** - 테스팅 프레임워크를 구동하기 위한 런처와 테스트 엔진을 위함 API 제공
- **JUnit Jupiter** - JUnit 5를 위한 테스트 API와 실행 엔진을 제공
- **JUnit Vintage** - JUnit 3, JUnit 4 로 작성된 테스트를 JUnit 5 플랫폼에서 실행하기 위한 모듈을 제공
``` xml
<dependencies>
	<dependency>
		<groupId>org.junit.jupiter</groupId>
		<artifactId>junit-jupiter</artifactId>
		<version>5.5.2</version>
		<scope>test</scope>
	</dependency>
</dependencies>
```
``` gradle
testImplementation("org.springframework.boot:spring-boot-starter-test") {
    exclude(group = "org.junit.vintage", module = "junit-vintage-engine")
}
```

> JUnit Jupiter, JUnit Vintage는 JUnit Platform의 구현체
> `spring-boot-starter-test` 를 사용하면  `assertJ`, `hamcrest`의 의존성도 같이 추가됨


Junit4 에서는 `public Class` 내부에 `public Method`만 실행 가능했지만
JUnit5 에서는 상관없이 `public`일 필요가 없음

### @BeforeAll, @AfterAll
여러 테스트가 모두 실행이 될 때 혹은 모두 실행 된 후 단 한번씩만 호출되는 메소드
반드시 `static void`를 사용하여 구현하여야함
> 접근제한자가 private, return 타입이 있으면 안됨
> JUnit4에서는 `@BeforeClass`, `@AfterClass`로 사용

### @BeforeEach, @AfterEach
여러 테스트가 각각 실행이 될때나 실행이 완료될때마다 실행되는 메소드
> JUnit4에서는 `@Before`, `@After`로 사용

### @Disabled
이 Annotation을 설정해놓으면 해당 테스트는 무시하고 넘어간다.
잠깐 깨진 테스트를 무시하고 넘어갈때 사용 하지만 해당 방법이 좋은 방법은 아니다.
> JUnit4에서는 `@Ignore`로 사용

## Junit5 주요 메서드

|메서드|설명|
|:-|:-|
|assertEquals(expected, actual)|실제값이 특정값과 같은지 검사|
|assertNotEquals(unexpected, actual)|실제값이 특정값과 같이 않은지 검사|
|assertSame(Object expected, Object actual)|두 객체가 동일한 객체인지 검사|
|assertNotSame(Object unexpected, Object actual)|두 객체가 동일하지 않은 객체인지 검사|
|assertTrue(boolean condition)|값이 true인지 검사|
|assertFalse(boolean condition)|값이 false인지 검사|
|assertNull(Object actual)|값이 null인지 검사|
|assertNotNull(Object actual)|값이 null이 아닌지 검사|
|fail()|테스트를 실패 처리한다.|
|assertThrows(Class<T> expectedType, Executable executable)|executable을 실행한 결과로 지정한 타입의 익셉션이 발생하는지 검사|
|assertDoesNotThrow(Executable executable)|executable을 실행한 결과로 익셉션이 발생하지 않는지 검사|


### 불필요한 연산 제거
``` java
assertEqual(StudyStatus.DRAFT, study.getStatus(), "스터디 " + StudyStatus.DRAFT + " 상태") // (1)

assertEqual(StudyStatus.DRAFT, study.getStatus(), () -> "스터디 " + StudyStatus.DRAFT + " 상태") // (2)
```
**(1)**에서 파라메터로 넘겨줄때마다 문자열 연산을 하여 `assertEqual` 함수로 넘겨주게 된다.
하지만 비슷하게 같아 보이는 **(2)** 예제에 대해서는 람다로 넘겨주게되어 에러가 나오는 상황에서만 문자열 연산이 발생하여 성능에 약간의 차이가 있다.
아주 복잡한 로직의 연산이라면 조금더 빠른 연산을 가져갈 수 있다.

### AssertAll을 이용하여 여러가지
```
    @Test
    @DisplayName("스터디 만들기")
    fun create_new_study() {
        val study = Study()
        assertNotNull(study) // (1)
        assertEquals(StudyStatus.DRAFT, study.status) // (2)
    }
```
위 예제에서 **(1)**번 상황에서 에러가 발생한다면 **(2)**코드는 실행이 안되어 한꺼번에 둘 다 에러가 났는지 체크하는것이 불가능하다.
이럴때 사용되는 함수가 `assertAll` 함수이다.
``` kotiln
    assertAll(
        { assertNotNull(study) },
        { assertEquals(StudyStatus.DRAFT, study.status) }
    )
```

### AssertTimeout
`Duration.ofMillis()` 메소드의 ms 값을 넘겨주어 타임아웃을 체크한다.
``` kotlin
assertTimeout(Duration.ofMillis(500)){
    Thread.sleep(1_000)
})
```
해당 메소드를 실행 시켰을 경우에는 1_000 만큼의 시간을 기다렸다가 실행이 끝나면 에러를 발생시킨다.

만약 메소드 실행 여부에 상관없이 설정한 시간이 지날경우 바로 에러를 발생시키고 싶다면 아래와 같이 실행 시킨다.
아래와 같이 실행시키면 해당 테스트는 500ms 뒤에 에러를 발생시킨다.
``` kotiln
assertTimeoutPreemptively(Duration.ofMillis(500)){
    Thread.sleep(1_000)
})
```
코드블럭을 별도의 스레드에서 실행하기 때문에(Thread.Local) 오동작을 일으킬 여지가 있다.
Spring Transactional의 기본전략이 Thread.Local로 사용하는데 다른 Thread에 공유가 되지 않기 때문에 롤백이 되어야 하는 상황에서 커밋이 될 상황이 생길 수 있음


## 환경에 따른 테스트 방법
### Assume
`assumeTrue`를 활용하여 해당 조건이 맞을때만 그 뒤에있는 테스트들을 수행한다.
조건이 맞지 않으면 해당 테스트를 건너뜀
``` kotlin
    @Test
    fun test_assume1() {
        val testEnv = System.getenv("TEST_ENV")
        assumeTrue("LOCAL".equals(testEnv, true))
        
        // 환경변수 TEST_ENV가 LOCAL이 아니라면 이 아래는 실행되지 않음
        val study = Study()
        assertNotNull(study)
        assertEquals(StudyStatus.DRAFT, study.status)
    }

    @Test
    fun test_assume2() {
        val testEnv = System.getenv("TEST_ENV")
        // 해당 환경에 따라 조건 설정 가능
        assumeTrue("LOCAL".equals(testEnv, true)) {
	        val study = Study()
	        assertNotNull(study)
	        assertEquals(StudyStatus.DRAFT, study.status)
		}
        
        assumeTrue("DEV".equals(testEnv, true)) {
	        val study = Study()
	        assertNotNull(study)
	        assertEquals(StudyStatus.ENDED, study.status)
		}
        
    }

```

### Enabled, Disabled

테스트 실행여부의 조건을 설정하는 어노테이션들이 있다.
- `@EnabledOnOs`, `@DisabledOnOs`
- `@EnabledOnJre`, `@DisabledOnJre`
- `@EnabledIfSystemProperty`, `@DisabledSystemProperty`
- `@EnabledIfEnvironmentVariable`, `@DisabledIfEnvironmentVariable`


``` java
Test
// OS가 일치할떄만 테스트 
@EnabledOnOS(OS.WINDOWS)
void windowTempPath() {
	...
}

@Test
// 자바 버전이 맞으면 테스트
@EnabledOnJre({JRE.JAVA_8, JRE.JAVA_9, JRE.JAVA_10, JRE.JAVA_11})
void testOnJre() {
	...
}

@Test
// 시스템 프로퍼티 값을 비교하여 테스트 실행 
@EnableIfSystemProperty(named = "java.vm.name", matchers=".*OpenJDK")
void openJdk() {
	...
}

@Test
// 환경변수 값을 비교하여 테스트 실행 
@EnableIfEnvironmentVariable(named = "java.vm.name", matchers=".*OpenJDK")
void openJdk() {
	...
}
```

## Tagging
@Tag를 이용하여

## 반복 테스트
### RepeatedTest
``` java
@DisplayName("반복 테스트")
@RepeatedTest(value = 10, name = "{displayName}, {currentRepetition} / {totalRepetitions}")
void repeatedTest(RepetitionInfo info) {
    System.out.println("test : " + info.getCurrentRepetition() + ", " 
        + info.getTotalCurrentRepetition()); 
}
```


### ParameterizedTest

``` java
@DisplayName("ParameterizedTest")
@ParameterizedTest(name = "{index} {displayName} message={0}")
@ValueSource(strings = {"value01", "value02", "value03"})
@EmptySource // ValueSource 이외에 message가 비어있는값이 한번 더 테스트 된다.
@NullSource // ValueSource 이외에 message가 Null 값이 한번 더 테스트 된다.
@NullAndEmptySource // EmptySource + NullSource
void parameterizedTest(String messege) {
    System.out.println(message)
}
```


## Test Instance
원래 JUnit의 메커니즘은 하나의 클래스안에 각각의 메소드별로 클래스의 인스턴스가 다르다.
그래서 전역변수를 변경해도 다른 테스트에서는 그 값을 공유하지 못한다
`TestInstance`를 선언해 주면 메소드들이 같은 인스턴스내에서 실행되게되어 전역변수 공유가 가능하다.

`@BeforeAll`, `@AfterAll`을 사용하고있으면 static 메소드가 아닌 일반 메소드로 사용한다.
``` java
@TestInstance(TestInstance.Lifecysle.PER_CLASS)
class TestClass() {
    
}
```

## Order
메소드의 실행 순서를 보장시켜준다.
옵션은 다음과 같이 3가지를 사용 가능하다
- Alphanumeric
- OrderAnnoation - method에 정의된 `@Order` annotation을 보고 순서를 결정
- Random

``` java
@TestInstance(TestInstance.Lifecysle.PER_CLASS)
@TestMethodOrder(MethodOrder.OrderAnnotation.class)
class TestClass() {
    
    @Order(1)
    void order01() {
        ...
    }
    
    @Order(2)
    void order02() {
        ...
    }
}
```


## JUnit Config
`test/resource` 경로에 `junit-platform.properties` 파일 생성하면 자동으로 설정됨
```
# 테스트 인스턴스 라이프사이클 일괄적용
junit.jupiter.testinstance.lifecycle.default = per_class

# 확장팩 자동 감지 기능
junit.jupiter.extensions.autodetection.enabled = true

# @Disabled 무시하고 실행하기
junit.jupiter.conditions.deactivate = org.junit.*DisabledCondition

# 테스트 이름 표기 전략 설정
junit.jupiter.displayname.generator.default = org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores
```

## 마이그레이션
JUnit5에서 `junit-vintage-engine` 의존성을 추가하면 JUnit4, JUnit3의 테스트 코드를 실행할 수 있다.
JUnit4에서 사용되는 `@Runwith`를 이제는 사용하지 않아도 된다.

`@Rule`은 기본적으로 지원하지 않지만, `junit-jupiter-migrationsupport` 모듈이 제공하는 `@EnableRuleMigrationSupport`를 사용하면 다음 타입의 Rule을 지원한다.
- ExternalResource
- Verifier
- ExpectedException

|JUnit4|JUnit5|
|:--|:--|
|@Category(Class)|@Tag(String)|
|@RunWith, @Rule, @ClassRule|@ExtendWith, @RegisterExtension|
|@Ignore|@Disabled|
|@Before, @After, @BeforeClass, @AfterClass|@BeforeEach, @AfterEach, @BeforeAll, @AfterAll|


