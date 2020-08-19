# Spring의 기본 구조

<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [Spring Container](#spring-container)
  - [Bean](#bean)
    - [Scope](#scope)
  - [BeanFactory](#beanfactory)
  - [Application Context](#application-context)
- [IoC / DI](#ioc--di)
- [PSA](#psa)
- [AOP](#aop)


## Spring Container
Spring Container, IoC Container 또는 Application Context라고 불리는 Spring Runtime Engine을 제공
Spring Container는 설정 정보를 참고로 해서 애플리케이션을 구성하는 오브젝트를 생성하고 관리
Stand Alone으로 동작 할 수 있지만 웹 모듈에서 동작하는 서비스나 서블릿으로 등록해서 사용.

### Bean
Spring에서는 Spring이 제어권을 가지고 직접 만들어 관계를 부여하는 오브젝트를 Bean 이라고 부른다.
Spring Bean은 Spring Container가 생성과 관계설정, 사용 등를 제어해주는 IoC가 적용된 오브젝트를 가리키는 말이다.

#### Scope
스프링에서 관리하는 Bean의 적용 범위를 설정 할 수 있다.
- singleton : 기본설정. 항상 같은 객체를 제공한다.
- prototype : 항상 다른 객체를 생성하여 제공한다.
- request : Http요청이 생길 때마다 새로 생성하여 제공된다.
- session : 웹의 세션에 따라 변경되어 제공

### BeanFactory
스프링의 IoC를 담당하는 핵심 컨테이너.
Bean을 등록, 생성, 조회, 리턴 그 외에 부가적인 빈을 관리하는 기능을 담당한다.
보통은 BeanFactory를 바로 사용하지 않고 ApplicationContext를 사용

### Application Context
Application Context는 `ApplicatoinContext` 인터페이스를 구현
BeanFactory를 확장하여 구현된 IoC 컨테이너
BeanFactory는 Bean의 생성과 제어의 관점에서만 이야기하고
ApplicationContext는 스프링이 제공하는 애플리케이션 지원 기능을 모두 포함하는 관점

ApplicationContext는 Bean Factory가 구현하는 `BeanFactory` 인터페이스를 상속했으므로 Application Context는 일종의 Bean Factory인 샘이다.

`@Configureation`이 붙은 객체는 Application Context의 설정 정보로 등록하고 `@Bean`이 붙은 메소드의 이름을 가져와  Bean목록을 만들어둔다.

## IoC / DI
오브젝트의 생명주기와 의존관계에 대한 프로그래밍 모델
스프링이 제공하는 모든 기술과 API, 심지어 컨테이너도 IoC/DI 방식으로 작성되어 있음

## PSA
서버, 특정 기술에 종속되지 않고 이식성이 뛰어나며 유연한 애플리케이션을 만들 수 있는데, 이를 가능하게 해주는것이 바로 PSA(portable service abstract)이다.
PSA는 특정 기술에 종속되지 않도록 유연한 추상 계층을 두는 방법이다.

## AOP
애플리케이션 코드 전반에 나오는 부가적인 기능을 독립적으로 모듈화하는 프로그래밍 모델
특정 클래스나 메소드의 시작점이나 마무리 지점에서 공통적인 코드를 실행시켜줌으로 다양한 엔터프라이즈 서비스를 적용하고도 깔끔한 코드를 유지할 수 있다.
