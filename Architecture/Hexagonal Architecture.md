# Hexagonal Architecture란

<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [기존 아키텍쳐에서 Hexagonal 까지의 여정](#기존-아키텍쳐에서-hexagonal-까지의-여정)
  - [기존 아키텍쳐의 문제점](#기존-아키텍쳐의-문제점)
  - [Layerd-Architecture에서의 변화](#layerd-architecture에서의-변화)
- [Hexagonal Architecture의 요소](#hexagonal-architecture의-요소)
  - [Port](#port)
  - [Adapter](#adapter)
- [Hexagonal의 장점](#hexagonal의-장점)
- [Package 구조](#package-구조)


Hexagonal Architecture로 알려져있는 `Ports & Adapters Architecture`는 2005년에 `Alistair Cockburn` 블로그에 소개되었다. 
그때 다음과 같은 문장으로 쇅되었다고 한다.
> Allow an application to equally be driven by users, programs, automated test or batch scripts, and to be developed and tested in isolation from its eventual run-time devices and databases. – Alistair Cockburn 2005, Ports and Adapters
> 
> 응용 프로그램이 사용자, 프로그램, 자동화 된 테스트 또는 배치 스크립트에 의해 동등하게 구동되고 최종 런타임 장치 및 데이터베이스와 분리되어 개발 및 테스트되도록합니다. – Alistair Cockburn 2005, Ports and Adapters

`Hexagonal Architecture`의 핵심 사항은 `Application을 외부 기술적인 부분들과 격리를 시켜야 한다` 입니다.

짧게 이야기 하자면 Application은 어떤 `Input`에서 들어왔고 어떤 `Output`으로 나가야 하는지에 대해서는 모르는것이 좋다. 

그래야 비즈니스 측면에서 변화가 생기더라도 Application 내부에서의 변화를 최소화 할 수 있기 때문이다.

이것의 궁극적인 목표는 Spring에서 핵심으로 여겨지는 PSA이다. (Portable Service Abstraction)
PSA는 Spring이 Tomcat에서 Netty로 변경을 한다고해도 우리는 수정해야 되는 코드들이 현저히 적거나 없을 수 있다.

`Hexagonal Architecture`의 목표는 바로 그렇게 기술과 도메인을 격리시켜 테스트하기 용이하게 만들고 바로 다른 기술로도 변경이 쉽게끔 만드는것이다.

## 기존 아키텍쳐에서 Hexagonal 까지의 여정
### 기존 아키텍쳐의 문제점
과거의 아키텍쳐는 FrontEnd와 BackEnd의 경계점이 모호한 경우가 다수 있다.

`front`에서 처리해야 할 로직을 `back-end`에서 일부 해준다던가 DTO 변경에 대한 로직을 Entity에서 처리하는 경우도 있으며 여러 다양한 End-Point에서 공통된 같은 자원을 사용하느라 리스코프치환원칙을 깨트리는 일들도 많이 발생한다. 
(A요청에서는 null을 허용하지만 A를 상속받은 B에서는 null을 허용하지 않는 케이스)

그리고 이와 반대로 DB에서 발생하는 문제점이 역으로 발생하는 문제도 있을 수 있다.
외부 라이브러리 객체 타입을 비즈니스로직에서 직접 사용한다거나. 비즈니스 로직에서 외부 라이브러리 타입을 참고하고 있는 경우이다.
외부 라이브러리에 대한 참조는 한쪽의 라이브러리에 편향되어 구현될 뿐만 아니라 자칫하다 라이브러리 변경이 있으면 전부 다시 구현해야 하는 큰 작업이 소요 될 수 있어 기피해야한다.

이와같이 기존의 아키텍쳐는 Input - Output 둘다의 큰 문제점을 가지고 있다.

### Layerd-Architecture에서의 변화
![Alt text](/asset/Architecture/ddd-layered.png)

2005년 `DDD`가 나오고나서 `system`과 관련이 깊은 것은 다름아닌 안쪽에 위치하고 있는 Layer라는 것을 알게되었고 Layer 계층의 최상층부와 최하층부의 Layer가 단순히 데이터들이 Application부터 나가고(exit point), Application으로 들어오는(entry point) 대칭성이 있는 관계라는게 밝혀졌다.

![Alt text](/asset/Architecture/hexagonal-arch-1.png)

일반적인 Layering Diagram과는 다르게 `Entry/Exit point`들을 각각 위와 다이어그램의 아래가 아니라 왼쪽과 오른쪽에 두었을때는 위와같이 나오게 된다.
그것을 이용하여 여러개가 `Entry/Exit point`들이 여러개가 존재한다는것을 다각형으로 표현하면 아래와 같은 이미지가 나오게 된다.
![Alt text](/asset/Architecture/hexagonal-arch-2.png)

여기에 비니지스 로직을 캡슐화 하고 각 `Entry/Exit point`에 대한 의존성을 끊어주고 그것들을 아답터로 처리하면 최종적인 `Hexagonal Architecture`가 그려지게된다.
![Alt text](/asset/Architecture/hexagonal-arch-3.png)

## Hexagonal Architecture의 요소
### Port
Port는 application 입장에서 consumer, 또는 application에서 나가거나/들어오는 끝 부분이라고 볼 수 있다.

많은 프로그래밍 언어에서 port는 interface로 표현된다. 예를 들어, search engine에서 검색을 수행하는 역할의 interface가 있을 수 있다. Application 비즈니스 로직에서 우리는 이 interface를 search engine의 구체적인 구현은 모른체 search engine을 사용하기 위한 Entry/Exit point로 사용할 것이다.

### Adapter
Adapter는 한 interface를 다른 interface로 바꿔주는 클래스를 말한다.
예를 들어, 한 adapter가 interface A를 구현하고 그 adapter에서 interface B를 주입받는다고 생각해보자. 그 adapter는 인스턴스화할 때 constructor에서 interface B를 구현한 객체를 주입받을 것이다. 외부에서는 interface A가 필요할 때 이 adapter를 주입한다. 그리고 ineterface A의 method가 호출될 때마다 adapter 내부의 interface B의 객체가 호출된다.

#### Primary Adapter
Primary Adapter 또는 Driving Adapter 라고 불리운다.
Driving Adapter에서 주로 application의 동작을 시작한다.
Driving Adapter에는 주로 UI 쪽이 들어간다.


#### Secondary Adapter
Secondary Adapter 또는 Driven Adapter라고 불리운다.
항상 Primary adapter에 의해서 반응하고 동작한다.


## Hexagonal의 장점
Application을 중심에 두고 외부적 요소와는 분리시켜 Application 내부가 캡슐화가 된다
외부적인 요소들을 분리시키면서 구체적인 구현체는 몰라도 쉽고 빠르게 테스트할 수 있고 다른 곳에서 재사용할 수 있다


## Package 구조



- com.domain.project
	- adapte
		- presentation (Controller)
			- cli
			- web
		- infrastructure (기술 영역)
			- elasticsearch
			- jpa
			- rebbitMQ
			- redis
		- configuration
		- service
			- account
			- project
	- application
	- domain.model
		- account
		- project



> 참고
> https://getoutsidedoor.com/2018/09/03/ports-adapters-architecture/
> https://blog.coderifleman.com/2017/12/18/the-clean-architecture/
> https://www.youtube.com/watch?v=TdyOH1xZpT8