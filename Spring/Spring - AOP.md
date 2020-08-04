#Study - Spring - AOP

<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [@Before](#before)
- [@After](#after)
- [@AfterReturning](#afterreturning)
- [@AfterThrowing](#afterthrowing)
- [@Around](#around)
- [AOP의 Order 설정](#aop의-order-설정)
- [@Pointcut 재활용](#pointcut-재활용)
- [Pointcut  Scope 정의](#pointcut--scope-정의)
- [Introduction](#introduction)
- [Weaving](#weaving)


Aspect-Oriented Programming

IoC 컨테이너에서 Aspect Annotation 기능을 활성화 하려면 구성 클래스 중 하나에 `@EnableAspectJAutoProxy`를 붙인다.
기본적으로 스프링은 JDK의 Dynamic Proxy를 생성하여 AOP를 적용하는데 인터페이스를 사용할 수 없거나 애플리케이션 설계상 사용하지 않을 경우엔 CGLIB으로 프록시를 만들 수 있다.
`@EnableAspectJAutoProxy`에서 `proxyTargetClasss=true` 속성을 설정하면 Dynamic Proxy 대신에 CGLIB을 사용한다.

Advice 종류
- `@Before`
- `@After`
- `@AfterReturning`
- `@AfterThrowing`
- `@Around`

JoinPoint에는 다음과 같은 정보들이 포함되어있다.
조인포인트 유형 (Spring AOP  메서드 실행만 해당), 메서드 시그니처(선언 타입 및 메서드명), 인수 값, 대상 객체, 프록시
``` kotlin
fun getJoinPointInfo(joinPoint: JoinPoint) {
    log.info("Join point kind : {}", joinPoint.kind)
    log.info("Signature declaring type: {}", joinPoint.signature.declaringTypeName)
    log.info("Signature name : {}", joinPoint.signature.name)
    log.info("Arguments : {}", joinPoint.args.toString())
    log.info("Target class : {}", joinPoint.target.javaClass.name)
    log.info("This class : {}", joinPoint.`this`.javaClass.name)
}
```
`joinPoint.target.javaClass.name` 은 원본 객체를 반환하며
`joinPoint.this.javaClass.name`은 프록시 객체를 반환한다.

## @Before
Before 어드바이스는 특정 프로그램 실행 지점 이전의 공통 관심사를 처리하는 메서드로 @Before를 붙이고 포인트컷 표현식을 어노테이션값으로 지정한다.
``` kotlin
Aspect
@Component
class CalculatorLoggingAspect {
    val log: Log = LogFactory.getLog(this::class.java)
	@Before("execution(* ArithmeticCalculator.add(..))")
//	@Before("execution(* *.*(..))")
    fun logBefore(joinPoint: JoinPoint) {
        log.info("The method ${joinPoint.signature.name}() begins with ${Arrays.toString(joinPoint.args)}")
    }
}
```
`* ArithmeticCalculator.add(...)`
앞부분의 와일드카드(`*`)는 모든 수정자 (public, protected, private), 모든 반환형을 매치함을 의미
인수 목록 부분에 쓴 두 점 (`..`)은 인수 개수는 몇개라도 좋다는 뜻

## @After
After 어드바이스는 조인포인트가 끝나면 실행되는 메서드로, @After를 붙여 표시
조인포인트가 정상 실행 또는 예외가 발생하더라도 실행을 보장.
``` kotlin
@After("execution(* ArithmeticCalculator.*(..))")
fun logAfter(joinPoint: JoinPoint) {
    log.info("The method ${joinPoint.signature.name}() ends")
}
```
## @AfterReturning
After와 달리 값을 반환할 경우에만 공통 관심사를 처리하고 싶으면 After Returning 어드바이스를 사용한다.
 조인포인트가 반환한 결과값을 가져오려면 @AfterReturning 의 returning 속성으로 지정한 변수명을 어드바이스 메서드의 인수로 지정한다.
``` kotlin
@AfterReturning(pointcut = "execution(* ArithmeticCalculator.*(..))", returning = "result")
fun logAfterReturning(joinPoint: JoinPoint, result: Any) {
    log.info("The method ${joinPoint.signature.name}() ends with $result")
}
```
## @AfterThrowing
조인포인트 실행 도중 예외가 날 경우에만 실행
메소드 Argument에 Type을 지정할 경우에는 해당 예외가 실행될 경우에만 발동된다.
``` kotlin
@AfterThrowing(pointcut = "execution(* ArithmeticCalculator.*(..))", throwing = "e")
fun logAfterThrowing(joinPoint: JoinPoint, e: Throwable) {
    log.error("exception $e has been thrown")
}

fun logAfterThrowing(joinPoint: JoinPoint, e: IllegalArgumentException) {
    log.error("IllegalArgumentException has been thrown")
}
```

## @Around
원본 조인포인트를 언제 실행할지, 실행 할지 말지,  반복하여 실행할지까지 제어가 가능
조인포인트의 인수형은 `ProceedingJoinPoint`로 고정되어있다.
`JoinPoint` 하위 인터페이스인 `ProceedingJoinPoint`를 이용하면 원본 조인포인트를 언제 진행할지 그 시점을 제어할 수 있다.

``` kotlin
@Around("execution(* *.*(..))")
fun logAround(joinPoint: ProceedingJoinPoint): Object throws Throwable {
	log.info("The method {}() begin with {}", joinPoint.signature.name, Arrays.toString(joinPoint.args));
	
	try {
		val result: Any = joinPoint.proceed();
		log.info("The method {}() end with", joinPoint.signature.name, result)
	} catch (e: IllegalArgumentException) {
		log.error("Illegal arlgument {} in {}()", Arrays.toString(joinPoint.args), joinPoint.signature.name)
	}
}
```


## AOP의 Order 설정
`CalculatorValidationAspect`와 `CalculatorLoggingAspect`의 우선 순위를 적용시킬 떄는 `Ordered`를 상속받아 Int를 리턴한다.
`getOrder()` 메서드가  반환하는 값이 작을수록 우선순위가 높다
``` kotlin
@Aspect
@Component
class CalculatorValidationAspect : Ordered {
	...
    override fun getOrder(): Int {
	    return 0
	}
}
@Aspect
@Component
class CalculatorLoggingAspect : Ordered {
	...
    override fun getOrder(): Int {
	    return 1
	}
}
```

``` kotlin
@Aspect
@Component
@Order(0)
class CalculatorValidationAspect { ... }
@Aspect
@Component
@Order(1)
class CalculatorLoggingAspect {	... }
```
`Order` 어노테이션을 이용하여 깔끔하게 구현 가능하다

## @Pointcut 재활용
Pointcut 표현식을 여러 번 되풀이해서 쓸 경우엔  어노테이션을 직접 써넣는 것보다 한번 표현식을 정해놓고 그것을 계속해서 재활용 하여 쓸 수 있다.
Aspect 포인트컷은 `@Pointcut`을 붙인 단순 메서드로 선언할 수 있다.
``` kotlin
@Aspect
class CalculatorPointcuts {
    @Pointcut("execute(* *.*(..)") // 이 부분을 재활용
    // 여러 Aspect에서 공유할 거면 public 형식으로 해야 한다.
    fun loggingOperation() {}
}

@Aspect
@Component
class CalculatorLoggingAspect {
    
    @Pointcut("execute(* *.*(..)") // 이 부분을 재활용
    private fun loggingOperation() {}

    @Before("loggingOperation()")
    fun logBefore(joinPoint: JoinPoint) { ... }

    @AfterReturning(pointcut = "loggingOperation()", returning = "result")
    fun logAfterReturning(joinPoint: JoinPoint, result: Any) { ... }

    @AfterThrowing(pointcut = "CalculatorPointcuts.loggingOperation()", throwing = "e")
    fun logAfterThrowing(joinPoint: JoinPoint, e: Throwable) { ... }
}
```

메서드끼리의 특이한 공통사항이 없을 경우에는 어노테이션으로 정의가 가능하다.
``` kotlin
@Target(ElementType.METHOD, ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface LoggingRequired {}

@LoggingRequired
class ArithmeticCalculatorImpl : ArithmeticCalculator {
	fun add(a: Double, b Double): Double { ... }
	fun sub(a: Double, b Double): Double { ... }
}

@Aspect
class CalculatorPointcuts {
	@Pointcut("annotation(com.example.demo.aop.LoggingRequired)")
	fun loggingOperation() {}
}
```

## Pointcut  Scope 정의


``` kotlin
execution(* com.example.demo.aop.calcurator.ArithmeticCalculator.*(..))

// 같은 패키지에 있다면 아래와 같이 줄여서 사용 가능하다.
execution(* ArithmeticCalculator.*(..))

// 해당 인스턴스의 모든 public 메소드
execution(public * ArithmeticCalculator.*(..))

// 해당 인스턴스의 모든 public 중 Double을 반환하는 메소드
execution(public Double ArithmeticCalculator.*(..))

// 첫번째 인수가 Double형인 메서드며 두점(`..`)으로 두 번째 이후 인수는 몇 개라도 상관없음
execution(public Double ArithmeticCalculator.*(Double, ..))

// 인수의 갯수와 형태를 구체적으로 명시 가능
execution(public Double ArithmeticCalculator.*(Double, Int))
```
``` kotlin
// 해당 패키지의 하위 모든것 메서드에 포인컷이 걸린다.
within(com.example.demo.aop.calcurator.*)

// 점 두개를 찍으면 하위 패키지까지 포인트컷이 걸린다.
within(com.example.demo.aop.calcurator..*)

// 한 클래스 내부에 구현된 메서드 실행 조인포인트를 매치
within(com.example.demo.aop.calcurator.ArithmeticCalculatorImpl)

// 같은 패키지 내부에서는 앞에 경로는 생략 가능
within(ArithmeticCalculatorImpl)

// 해당 인터페이스를 구현한 모든 인스턴스에 포인트컷 매치
within(ArithmeticCalculator+)

// 포인트컷 조합
within(ArithmeticCalculator+) || within(UnitCalculator+)
```

``` kotlin
// 포인트컷 조합 및 Argument 명세화
@Before("execution(* *.*(..)) && target(target) && args(a, b)")
fun logParameter(target: Any, a: Double, b: Double) {
	log.info("Target class : ${target.javaclass.name}")
	log.info("Arguments : $a, $b")
}

// 포인트컷에서도 사용 가능
@Pointcut("execution(* *.*(..)) && target(target) && args(a, b)")
fun parameterPointcut(target: Any, a: Double, b: Double) {}

@Aspect
class CalculatorLoggingAspect {
	@Before("CalculatorPointcuts.parameterPointcut(target, a, b)")
	fun logParameter(target: Any, a: Double, b: Double) {}
}
```

## Introduction
여러개의 클래스에서 유용한 기능들을 끌어들여 하나의 Util성 클래스를 구현 할 수 있다.
다중상속 같은 구현이 가능
``` kotlin
interface MaxCalculator {
    fun max(a: Double, b: Double): Double
}
class MaxCalculatorImpl : MaxCalculator {
    override fun max(a: Double, b: Double) = if (a > b) a else b
}
interface MinCalculator {
    fun min(a: Double, b: Double): Double
}
class MinCalculatorImpl : MinCalculator {
    override fun min(a: Double, b: Double) = if (a < b) a else b
}
```

Introduction을 이용하여 기존 객체에 새로운 상태를 추가해서 호출 횟수, 최종 수정일자 등 사용 내역을 파악하고 싶은 경우를 해결 할 수 있다.
```
interface Counter {
    var count: Int
    fun increase()
}

class CounterImpl(override var count: Int = 0) : Counter {
    override fun increase() {
        count++
        println("count : $count")
    }
}

@Aspect
@Component
class CalculatorIntroductionControl {
    @DeclareParents(
        value = "com.example.demo.aop.calculator.*CalculatorImpl",
        defaultImpl = CounterImpl::class)
    lateinit var counter: Counter

    @After("execution(* com.example.demo.aop.calculator.*Calculator.*(..)) && this(counter)")
    fun increaseCount(counter: Counter) {
        counter.increase()
    }
}
```
`Calculator` 글자가 들어가는 객체 내부의 메소드가 호출될때마다 count가 1씩 올라간다.

## Weaving
Aspect를 대상 객체에 적용하는 과정
Spring AOP는 런타임에 동적 프록시를 활용해 위빙을 하는 방면 AspectJ 프레임워크는 compile-time, load-time 모두 지원한다.
> AspectJ에서 컴파일 타임은 `ajc`라는 전용 컴파일러가 담당.

- 컴파일 타임 위빙 (compile-time weaving)
	- Aspect를 자바소스 파일에 엮고 위빙된 바이너리 클래스 파일을 결과로 내놓음
	- 이미 컴파일 된 클래스 파일이나 JAR파일 안에도 Aspect를 강제로 주입할 수 있다.  이를 **포스트 컴파일 타임 위빙** (post-compile-time weaving) 이라고 한다.
	- 컴파일 시점 위빙, 포스트 컴파일 타임 위빙 모두 클래스를 IoC 컨테이너에 선언하기 이전에 수행 할 수 있으며 스프링은 위빙 과정에 전혀 관여하지 않는다.

- 로드 타임 위빙 (load-time-weaving)
	- JVM이 클래스 로더를 이용해 대상 클래스를 로드하는 시점에 일어남
	- 바이트코드에 코드를 넣어 클래스를 위빙하려면 특수한 클래스 로더가 필요.
	- AspectJ, Spring 둘 다 클래스 로더에 위빙 기능을 부여한 로드 타임 위버를 제공.
	- 로드타임 위버는 간단한 설정으로 바로 사용 가능,.