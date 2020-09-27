# Spring Boot - Properties 정리
## 개요
Spring에서 properties는 설정 중 가장 기본적이면서 또한 가장 자주 들어오는 질문중 하나입니다.
properties를 설정하는 방법들이 너무나도 많고 어떻게 여러가지 방법중 하나씩만 설명되어 있는 경우들이 많아 사용하기에 많은 검색들이 필요하기도 합니다.
그래서 많은 사람들이 질문하며 궁금해하던 내용들과 사용방법, 자주 사용되는 패턴들을 모아서 블로깅을 한번 해볼까하여 정리합니다.

>해당 샘플 코드들은 코틀린에 스프링 부트(`2.3.1.RELEASE`)를 기본으로 하고있습니다.
>잘못된 정보가 있거나 추가적인 정보를 원하시는 경우에는 이메일이나 댓글 부탁드립니다.

## 사용법
### 기본적인 사용법

``` shell
# application.properties
say-hello=Hello World!
```
``` kotlin
// HelloController.kt
@RestController
class HelloController {

    @Value("\${say-hello}")
    lateinit var sayHello: String

    @Value("\${say-hello: default hello message}")
    lateinit var defaultSayHello: String

    @GetMapping("/hello")
    fun hello(): String {
        return "say : $sayHello"
    }
}
```

위 코드와 같이 기본적인 컨트롤러에서 필드 변수에 `@Value`를 이용하여 properties에 등록되어있는 값을 불러와서 출력하고있습니다.
만약 properties에 지정된 값이 없을 경우 `Caused by: java.lang.IllegalArgumentException: Could not resolve placeholder 'say-hello' in value "${say-hello}"` 에러를 내게되며 `@Value` 어노테이션 프로퍼티의 키값 설정하는 문자열 끝에 `:`을 이용하여 기본 값 지정이 가능합니다. (defaultSayHello 변수 참조)

> kotlin은 문자열 내에서 `$` 기호가 문자열 템플릿을 사용하는것을 나타냄으로 앞에 `\`를 붙여 escape 처리를 해줘야합니다.
> ``` java
// java에서 문자열 합쳐서 보여주는 방법
String name = "Java";
System.out.println("Hello, " + name + "!");
```
> ``` kotlin
// Kotlin String Templates
val name = "Kotlin";
println("Hello, $name");
```

### Properties 사용시 알아야할 Spring 기본 지식

properties를 사용하기 이전에 **필수로 알아야 할 내용**으로는 Spring이 자동으로 값을 넣어주거나 의존성을 주입해주는것을 **DI(의존성 주입)**이라고 하며 의존성 주입을 받기 위해서는 애플리케이션 소스 코드가 아닌 독립된 컨테이너가 객체의 생성, 사용, 소멸에 대한 제어권을 받아서 처리하도록 하여야 합니다.
이를 제어의 역전(IoC)라고 불리우며 해당 컨테이너를 **IoC 컨테이너**라고 부릅니다.

IoC 컨테이너 내부에 객체를 등록하기 위해서는 Bean이라는 객체로 만들어주어야 하는데 Bean객체를 만들기 위해서는 stereotype이라고 불리우는 어노테이션을 객체위애 명시해주어야 합니다.

> 가장 널리 사용되는 stereotype으로는 `@Component`, `@Controller`, `@Service`, `@Repository`가 있습니다.

stereotype 이외에도 직접 `@Bean` 객체로 등록하거나 `@Configuration`으로 등록해주는 방법도 있습니다.

실제로 많은 분들이 `@Value`를 사용하는 객체를 Bean으로 등록해주지 않아 동작이 되지않는 경험을 많이하고계십니다.

### List
properties에서 리스트 형태를 나타내는 방법은 `,`를 이용하여 처리가 가능합니다.
당연한 결과겠지만 문자열 마지막에 쉼표가 있는 경우에 마지막 리스트 요소는 빈 문자열이 됩니다.
``` shell
# application.properties
test.color-list=red, blue, green, white, black
```
``` kotlin
@Value("\${test.color-list}")
lateinit var colorList: List<String>

println(colorList.toString())
// [red, blue, green, white, black]
```


### Map

properties에서 리스트 형태를 나타내는 방법은 JSON형태로 작성하여 처리가 가능합니다.
아래와 같은 형식으로 리스트로 받을 수 있습니다.

``` shell
# application.properties
test.profile={name: "ssabae", email: "llsb156@gmail.com"}
```

``` kotlin
@Value("#{\${test.profile}}")
lateinit var profile: Map<String, String>

println(profile)
// {name=ssabae, email=llsb156@gmail.com}
```
> `@Value`를 `${test.profile}` 형식으로 불러오면 문자열 형태로 읽어오기 때문에 `#{...}` 형태로 한번 더 감싸주어 맵 형태로 변경하기위해 추가적인 선언이 필요합니다.


### random value

스프링 properties에서는 별도의 설정없이 랜덤값 사용이 가능합니다.
랜덤값은 `@Value`로 값을 할당 받는 즉시에 결정되며 하나의 class에서 동일한 properties의 랜덤값을 할당 받아도 모든 값이 다 다르게 적용이 됩니다.

``` shell
// application.properties
test.secret=${random.value}
test.number=${random.int}
test.bignumber=${random.long}
test.uuid=${random.uuid}
test.number.less.than.ten=${random.int(10)}
test.number.in.range=${random.int[1024,65536]}
```

``` kotlin
@Value("\${test.secret}")
var secret1: String = ""

@Value("\${test.secret}")
var secret2: String = ""

@Value("\${test.secret}")
var secret3: String = ""

@Value("\${test.number}")
var number: Int = 0

@Value("\${test.bignumber}")
var bignumber: Long = 0

@Value("\${test.uuid}")
var uuid: UUID = UUID.randomUUID()

@Value("\${test.number.less.than.ten}")
var lessThan: Int = 0

@Value("\${test.number.in.range}")
var range: Int = 0

println("""
    secret1 : $secret1
    secret2 : $secret2
    secret3 : $secret3
	number : $number
	bignumber : $bignumber
	uuid : $uuid
	lessThan : $lessThan
	range : $range
""".trimIndent())

// secret1 : 9c5b92ddf762a7bd1955ad5c02633f9d 
// secret2 : c7233ad147a6604e06e03d3b917bcef0 
// secret3 : cd6257f1bca21e96ca2ad824e7828d63
// number : 994852137 
// bignumber : -7962814938015041999 
// uuid : 48864b5b-df51-4881-8d10-ace41e6d478a 
// lessThan : 2 
// range : 32937
```
> `PropertySource<Random>`을 상속받은 `RandomValuePropertySource` Bean에서 properties 내부에 `${random.` prefix를 가진 항목들을 찾아서 사용자가 원하는 형태대로 변형하여 전달해 주는 역할을 합니다.



### file, classpath, http

``` kotlin
@Value("classpath:banner.txt")
private Resource banner01;

@Value("file:c:/shop/banner.txt")
private Resource banner02;

@Value("classpath:com/example/spring/banner.txt")
private Resource banner03;

@Value("http://springrecipes.apress.com/shop/banner.txt")
private Resource banner04;
```
위 예제처럼 직접적인 경로를 지정하여 외부나 프로젝트 내부에 있는 파일을 Access할 수 있으며 http를 통하여 외부에있는 값을 받을수도 있습니다.




## 객체화하여 사용하기
`application.properties`가 아닌 기타 설정(데이터 소스, 계정정보, 특정 모듈 외부 입력 정보 등) 파일을 따로 분리하여 사용할 때가 많습니다.
그럴 경우에는 자동으로 properties가 등록되지 않아 사용할 수 없는데 이 정보들을 객체화 하여 사용하는 방법들 중 `@PropertySource`, `@ConfigurationProperties`에 대해서 알아보겠습니다.

### @PropertySource
`application.properties`가 위치하는 `classpath`에 `app-info.properties` 파일을 하나 생성하여 app에 관련된 정보를 넣어주어 객체로 사용해 보겠습니다.
``` shell
# app-info.properties
app.name=ssabae velog
app.description=this is ssabae's velog
app.url=https://velog.io/@lsb156
```
``` kotlin
// AppInfo.kt
@Component
@PropertySource("classpath:app-info.properties")
class AppInfo {

    @Value("\${app.name}")
    lateinit var name: String

    @Value("\${app.description}")
    lateinit var description: String

    @Value("\${app.url}")
    lateinit var url: String

    override fun toString(): String {
        return "AppInfo(name='$name', description='$description', url='$url')"
    }
}
```
`@PropertySource` 어노테이션 내부에 사용할 프로퍼티 위치를 입력하여 준뒤에 반드시 `@Component`로 등록하여 스프링에서 빈으로 관리되게 해주어야합니다.

```kotlin
@Autowired
lateinit var appInfo: AppInfo

println(appInfo)
// AppInfo(name='ssabae velog', description='this is ssabae's velog', url='https://velog.io/@lsb156')
```
> Spring Boot에서 PropertySource의 우선순위는 다음과 같습니다.
> (밑으로 갈수록 우선순위가 낮아집니다)
``` list
1. Devtools global settings properties on your home directory (~/.spring-boot-devtools.properties when devtools is active).
2. @TestPropertySource annotations on your tests.
3. @SpringBootTest#properties annotation attribute on your tests.
4. Command line arguments.
5. Properties from SPRING_APPLICATION_JSON (inline JSON embedded in an environment variable or system property)
6. ServletConfig init parameters.
7. ServletContext init parameters.
8. JNDI attributes from java:comp/env.
9. Java System properties (System.getProperties()).
10. OS environment variables.
11. A RandomValuePropertySource that only has properties in random.*.
12. Profile-specific application properties outside of your packaged jar (application-{profile}.properties and YAML variants)
13. Profile-specific application properties packaged inside your jar (application-{profile}.properties and YAML variants)
14. Application properties outside of your packaged jar (application.properties and YAML variants).
15. Application properties packaged inside your jar (application.properties and YAML variants).
16. @PropertySource annotations on your @Configuration classes.
17. Default properties (specified using SpringApplication.setDefaultProperties).
```


### @ConfigurationProperties
`properties`파일에서 `prefix`가 동일한 설정 정보들을 찾아 객체로 자동으로 변환해 주는 기능을 사용하려면 `@ConfigurationProperties`에 `prefix` 이름을 넣어주어 Type-Safe하게 설정파일을 객체화 시킬 수 있습니다.
``` shell
#application.properties
app.name=ssabae velog
app.description=this is ssabae's velog
app.url=https://velog.io/@lsb156
app.user.username=admin
app.user.password=password
app.user.roles=ADMIN, USER
```
``` kotlin
@ConfigurationProperties("app")
class AppInfo {

    lateinit var name: String

    lateinit var description: String

    lateinit var url: String

    var user: UserInfo = UserInfo()

    companion object {
        class UserInfo {
            lateinit var username: String
            lateinit var password: String
            lateinit var roles: Set<String>

            override fun toString(): String {
                return "UserInfo(username='$username', password='$password', roles=$roles)"
            }
        }
    }

    override fun toString(): String {
        return "AppInfo(name='$name', description='$description', url='$url', user=$user)"
    }
}
```

```kotlin
@Autowired
lateinit var appInfo: AppInfo

println(appInfo)
// AppInfo(name='ssabae velog', description='this is ssabae's velog', url='https://velog.io/@lsb156', user=UserInfo(username='admin', password='password', roles=[ADMIN, USER]))
```
`@ConfigurationProperties` 어노테이션이 붙어있는곳에 `@ConstructorBinding`를 같이 선언하여 생성자를 기본으로 사용하게 설정 할 수 있으며 선언되지 않는 프로퍼티로 인해 null이 나올 수 있는 상황에 대해서는 생성자의 인자값에 `@DefaultValue`를 이용하여 기본 값을 주입 할 수 있습니다.
``` kotlin
@ConstructorBinding
@ConfigurationProperties("app")
class AppInfo(
    var name: String,
    var description: String,
    var url: String,
    @DefaultValue("default value")
    var emptyValue: String,
    @DefaultValue
    var user: UserInfo?
) {

    companion object {
        class UserInfo(
            var username: String,
            var password: String,
            var roles: Set<String>
        ) {
            override fun toString(): String {
                return "UserInfo(username='$username', password='$password', roles=$roles)"
            }
        }
    }

    override fun toString(): String {
        return "AppInfo(name='$name', description='$description', url='$url', emptyValue='$emptyValue', user=$user)"
    }
}
```
무엇보다 가장 중요한것은 `@Configurate` 혹은 `Application` Class에 `@EnableConfigurationProperties(AppInfo::class)`로 등록을 해줘야 사용이 가능합니다.

## Environment를 이용한 사용법
Environment 객체를 직접 호출하여 객체 내부의 필드에 값을 할당 하는 방법도 있습니다.
`environment.getProperty`의 인자값에 Type과 default 값이 바로 설정가능하여 Type-safe하게 처리가 가능합니다.

``` shell
app.name=ssabae velog
app.description=this is ssabae's velog
app.url=https://velog.io/@lsb156
```

``` kotlin
@Component
class AppInfo(
    var environment: Environment
) {

    lateinit var name: String
    lateinit var description: String
    lateinit var url: String
    var version: Int = 0

    @PostConstruct
    fun preConstructor() {
        name = environment.getProperty("app.name") as String
        description = environment.getProperty("app.description") as String
        url = environment.getProperty("app.url") as String
        // properties에 정의되어있지 않은 항목에 기본값을 넣어주어 오류 방지
        version = environment.getProperty("app.version", Int::class.java, 10)
    }

    override fun toString(): String {
        return "AppInfo(name='$name', description='$description', url='$url', version=$version)"
    }
}
```


## Profile에 따른 분기법
각 환경에 따라 prifile 값을 다르게하여 properties 파일 분기 처리가 가능합니다.
```
# application.properties
app.name=ssabae velog
app.description=this application name is ${app.name}

# application-prod.properties
app.name=[prod] velog

# applicatoin-dev.properties
app.name=[dev] velog
```
spring profile에 `dev`가 active 되어있다면 `app.name` 값을 `applicatoin-dev.properties`에서 가저옵니다.
그리고 `app.description`값을 `applicatoin-dev.properties`에서 먼저 찾고 없으면 기본 properties인 `application.properties`에서 찾게 됩니다.

스프링 부트에서는 자동으로 `application-{profile}.properties` 패턴으로 properties를 찾아서 적용시켜줍니다.

이를 이용하여 실제 서비스에서 사용하는 데이터베이스 아이디 혹은 비밀번호가 들어있는 `application-prod.properties` 파일을 git에 올리지 않고 따로 실서버에 업로드하여 해당 프로필만 로드하여 사용하도록 구성이 가능합니다.

각 `Local`, `Dev`, `Stage`, `Product` 환경별로 profile을 설정해놓으면 일일이 변경하거나 깜빡하고 수정하지 않아서 나중에 큰 에러가나는 수고를 덜 수 있습니다.

## 마무리
properties가 지원해주는 기능은 본 블로그에 나와있는것보다 훨씬 많습니다.
우선순위가 되는것도 예상한것보다 많았는데 이번에 확실하게 알수 있어서 좋은 기회가 된것 같습니다.
개발자들이 자주 사용할만한 것들을 간추려서 정리를 하였고 시간이 되면서 점점 보강해 나가거나 새로운 글로 올리도록 하겠습니다 (가능하면 YAML, UnitTest에 대해서 정리하겠습니다.)



> 참고내용
> [Spring Boot Features - Externalized Configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config)
> Spring 5 레시피


