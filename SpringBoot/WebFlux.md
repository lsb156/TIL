# WebFlux
리액티브 마이크로서비스는 마이크로서비스의 한단계 진화한 모습이다.
리액티브 패러다임을 기반으로 기존의 전통적인 아키텍처를 능가하는 대응력, 복원성, 탄력적인 메시지 기반 서비스를 제공하는 것을 목표로 한다.

<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [Netty](#netty)
- [리액티브 서비스 만들기](#리액티브-서비스-만들기)
  - [Publish & Subscribe](#publish--subscribe)
    - [back-pressure](#back-pressure)
  - [단일객체 게시](#단일객체-게시)
  - [여러 객체 게시](#여러-객체-게시)
  - [RouterFunction 사용](#routerfunction-사용)

## Netty
네티는 원래 넌블로킹 IO 작업을 수해앟ㄹ 수 있게 하는 Client-Server 프레임워크로 만들려는 아이디어를 가진 JBoss에 의해 개발되었다,
이런 기능을 위해 Reacotr 패턴의 메시지 기반 구현을 사용한다.
네티는 HTTPS, SSL/TLS 또는 DNS 같은 주요 알고리즘 및 프로토콜을 지원하지만 HTTP/2, Websocket, Google Protocol Butffer 같은 최신 프로토콜도 지원한다.

스프링 부트 1.X 는 아파치 톰캣을 기본 어플리케이션/웹 서버로 사용하지만 톰캣은 블럭킹 작업만 지원한다.
스프링 부트 2.0에서는 넌블로킹 IO 기능의 리액티브 서비스를 위해 네티를 대신 선택했다.

이 기술을 선택하면 마이크로서비스는 더 많은 부하를 처리할 수 있으며 그 어느때보다 효과적이 될 것이다.
> NodeJS, Nginx, Apache Mina, Vert.X, Akka들도 넌블럭킹 IO 시스템이다.

## 리액티브 서비스 만들기
### Publish & Subscribe
리액티브 프로그래밍은 일련의 이벤트가 감지되면 필요한 사용자에게 전송되는 이벤트모델(Event Model) 매커니즘을 기반으로 한다.
 이벤트를 수신하려고 할 때, 정의하는 것이 구독자(Subscriber)이다. 구독자는 이벤트가 생성될 때 이벤트를 수신해야 하는 책임이 있는 객체다.
 이벤트를 발생시키는 책임이 있는 객체를 게시자(Publisher) 라고한다.
 게시자와 구독자 객체는 결과를 기다리는 작업을 블로킹하지 않는다.
 이런 객체는 필요할 때 호출되는 수신기(Listener)이다.
 스프링은 서비스를 사용하는 모든 사용자에게 데이터를 보내기 위해 이벤트를 구독한다,
 이 데이터는 리액티브 스트림 사양에 정의된 대로 리액티브 스트림으로 전송되어 넌블로킹의 Back-pressure를 제공한다.
#### back-pressure
 너무 커서 처리하기 힘든 데이터 Stream을 조절하는 것
 back-pressure를 하는 목적은 Subscriber가 실질적으로 처리할 수 있을 만큼의 수준으로 데이터를 제공받는 것
 Publisher 쪽에서 처리되지 않는 데이터는 버퍼가 되기도 한다.
 pull-push hybrid approach to data stream - publisher가 처리 할 만큼의 수준만 요청 받고 다 처리하면 다시 요청 받는 형식
 Deley 가 발생해서 Latency가 발생할 수 있고 메모리를 다 써버려 Out of Memory가 발생 할 수도있다.

### 단일객체 게시
리액터는 모노(Mono)라는 클래스를 통해 리액티브 게시자를 정의하는 방법을 제공하지만, 이 게시자는 하나의 결과만 보낼 수 있다.

``` kotlin
// 일반적인 Mono 생성
val customerMono = Mono<Customer> = Mono.just(Customer(1, "Mono"))
// 고차함수 
val customerMono = Mono<Customer> = Customer(1, "Mono").toMono()
// 타입 추론
val customer = Customer(1, "Mono").toMono()
```
모노는 실제로 우리가 만든 인스턴스가 아니라 앞으로 얻으려고 하는 것에 대한 약속이다.
`Mono<Customer>`로 선언할 떄는 이 게시자가 앞으로 Customer를 게시할 것임을 나타낼 뿐이다.
누군가가 해당 게시자를 등록하면 데이터를 얻게 된다.

### 여러 객체 게시
리액터는 0에서 무한대의 요소를 가진 게시자를 만들 수 있는 클래스를 제공한다.
클래스의 이름은 Flux이다.
단순 플럭스를 만들려면 다음 작업을 수행한다.
``` kotlin
val customer = Flux.fromIterable(listOf(Customer(1, "One"), Customer(2, "two")))
// like kotlin
val customerFlux = listOf(Customer(1, "One"), Customer(2, "two")).toFlux()
```
Flux도 Mono와 마찬가지로 구독 시 실행될 수 있다는 약속이다.
Controller가 Flux를 반환하면 스프링은 자동 구성으로 새로운 요청이 들어오면 구독하게 된다.
> 모노는 결과를 하나만 반환해야 하는 경우에 사용할 수 있지만 플럭스는 결과를 O-n개 반환 하는 경우에 사용할 수 있다. 하지만 하나만 반환하려면 플럭스 대신 모노를 사용하는 것이 더 좋다.

### RouterFunction 사용
Annotation기반 시스템ㄴ과 마찬가지로 마이크로서비스로 들어오는 요청을 어떻게 처리할지 정의하기 위해 컨트롤러 대신 RouterFunction을 사용한다.
``` kotlin
// CustomerRouter.kt
@Component
class CustomerRouter(private val customHandler: CustomHandler) {
    @Bean
    fun customerRoutes(): RouterFunction<*> = router {
        "/functional".nest {
            "/customer".nest {
                GET("/{id}", customerHandler::get)
                POST("/", customerHandler::create)
            }
            "/customers".nest {
                GET("/", customerHandler::search)
            }
        }
    }
}

// CustomHandler.kt
@Component
class CustomerHandler(val customerService: CustomerService) {
    fun get(serverRequest: ServerRequest): Mono<ServerResponse> =
        customerService.getCustomer(serverRequest.pathVariable("id").toInt())
            .flatMap { ok().body(fromObject(it)) }
            .switchIfEmpty(notFound().build())

    fun search(serverRequest: ServerRequest) =
        ok().body(
            customerService.searchCustomers(
	            serverRequest.queryParam("nameFilter").orElse("")),
            Customer::class.java
        )

    fun create(serverRequest: ServerRequest) =
        customerService.createCustomer(serverRequest.bodyToMono()).flatMap {
            status(HttpStatus.CREATED).body(fromObject(it))
        }.onErrorResume(Exception::class) {
            badRequest().body(
                fromObject(
                    ErrorResponse(
                        "error createing customer",
                        it.message ?: "error"
                    )
                )
            )
        }
}

// CustomerServiceImpl.kt
@Component
class CustomerServiceImpl : CustomerService {
    companion object {
        val initialCustomers = arrayOf(
            Customer(1, "Kotlin"),
            Customer(2, "Spring"),
            Customer(3, "MicroService", Telephone("+44", "198273128"))
        )
    }

    val customers = ConcurrentHashMap<Int, Customer>(initialCustomers.associateBy(Customer::id))

    override fun getCustomer(id: Int) = customers[id]?.toMono() ?: Mono.empty()

    override fun searchCustomers(nameFilter: String): Flux<Customer> =
        customers.filter {
            it.value.name.contains(nameFilter, true)
        }.map(Map.Entry<Int, Customer>::value).toFlux()

    override fun createCustomer(customerMono: Mono<Customer>): Mono<Customer> =
        customerMono.map {
            customers[it.id] = it
            it
        }.toMono()
}
```
위와 같이 정의하면 라우터가 `/functional` 경로의 모든 요청을 처리한다.
> 이 라우터 기능을 만들기 위해 스프링이 웹플럭스를 생성하는 코틀린 DSL(Domain Specific Language)을 사용한다,
https://kotlinlang.org/docs/reference/type-safe-builders.html

- Router : 리액티브 서비스가 응답하는 경로와 메소드를 처리
- Handler : 궤적인 요청을 응답으로 변환하는 로직을 수행
- Service : 도메인의 비즈니스 로직을 캡슐화

이런 별도의 레이어가 있으면 새 기능을 추가할 필요가 있는 곳을 변경할 때 도움이 된다.
라우터는 다른 기능을 위해 동일한 핸들러를 호출할 수 있으며, 핸들러는 여러 서비스를 결합할 수 있다. 한 레이어를 변경해도 다른 레이어에 영향을 주지 않을 것이다. 예를 들어, 서비스에서 도메인 로직을 변경하더라도 라우터 또는 핸들러를 변경할 필요가 없다.
> 마이크로 서비스에서 커플링(Coupling)을 방지해야 하는것과 마찬가지로 레이어를 만들 때도 커플링을 피해야 한다. 마이크로 서비스를 만들 때 이런 레이어와의 상호 작용 방식을 생각하자. 단일 책임 원칙(Single Responsibility Principle)을 적용해야 한다.
> 하나의 레이어의 변화에는 오직 한 가지 이유만 있어야 한다.

