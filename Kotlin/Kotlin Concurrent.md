


## CPU 바운드와 I/O 바운드
병목현상은 다양한 유형의 성능저하가 발생하는 지점을 나타낸다.
동시성과 병렬성이 CPU나 I/O 연산에 바인딩 되었는지의 여부에 다라 알고리즘의 성능에 영향을 끼친다

### CPU 바운드
싱글 코어에서 한가지의 작업을 3개의 스레드로 나누어 진행한다면 context switch 비용이 발생하기 때문에
하나의 스레드에서 작업하는것보다 더 오랜 시간이 소요된다.
그래서 CPU에서 작업되는 스레드의 갯수를 정할때는 코틀린의 스레드풀인 `CommonPool`를 사용하여 작업한다.
> CommonPool의 크기는 머신의 코어 수에서 1을 밴 값이ㄴ다 4개의 코어가 있는 머신에서는 `CommonPool`의 크기가 3이 된다.

### I/O 바운드
I/O 바운드 알고리즘은 끊임없이 무엇인가를 대기하는 상태이다.
단일코어 기기에서는 대기하는 중에 다른 유용한 작업에 프로세스를 사용할 수 있도록 한다.
따라서 I/O 바운드인 동시성 알고리즘은 병렬(멀티코어)이거나 단일코어에 상관없이 유사하게 수행된다.

## 동시성의 어려움
### Race condition (경합 조건)
코드를 동시성으로 작성했지만 순차적 코드처럼 동작할 것이라고 예상할 때 발생
A에서 항상 B의 작업이 일찍 끝날거라 생각하고 A가 끝나자마자 B 결과에 접근할때 발생
``` kotlin
fun main(args: Array<String>) = runBlocking {
    asyncGetUserInfo(1)
    // Do some other operations
    delay(1000)
    println("User ${user.id} is ${user.name}") // UninitializedPropertyAccessException
}

fun asyncGetUserInfo(id: Int) = GlobalScope.async {
    delay(1100)
    user = UserInfo(id = id, name = "Susan", lastName = "Calvin")
}
```

### Atomic operations (원자성 위반)
동시에 두가지의 쓰레드에서 한가지의 자원에 접근할때에는 원자성이 깨지게 된다.
A, B 쓰레드에서 동시에 count값을 읽고 1씩 더한다고하면
A는 0을 읽고 B도 0을 읽고 둘다 count를 1로 덮어쓰게된다.
그래서 아래 코드는 대부분 2100보다 적게 나온다.
``` kotlin
var counter = 0

fun main() = runBlocking {
    val workerA = asyncIncrement(2000)
    val workerB = asyncIncrement(100)

    workerA.await()
    workerB.await()

    print("counter [$counter]")
}

fun asyncIncrement(by: Int) = GlobalScope.async {
    for (i in 0 until by) {
        counter++
    }
}
```

### Dead Lock (교착상태)
동시성 코드가 올바르게 작동하려면 다른스레드에서 작업이 완료되는 동안 실행을 일시 중단 하거나 차단할 필요가 있다.
이러한 복잡성 때문에 순환적 의존상태 (circular dependencies)가 드물지 않게 발생한다.
Race condition과 같이 자주 발생한다.
``` kotlin
lateinit var jobA : Job
lateinit var jobB : Job

fun main(args: Array<String>) = runBlocking {
    jobA = launch {
        delay(1000)
        // wait for JobB to finish
        jobB.join()
    }

    jobB = launch {
        // wait for JobA to finish
        jobA.join()
    }

    // wait for JobA to finish
    jobA.join()
    println("Finished")
}
```

### Live Lock (라이브 락)
Live Lock은 어플리케이션이 올바르게 실행을 계속할 수 없을 때 발생하는 교착 상태와 유사  
애플리케이션의 상태는 지속적으로 변하지만 애플리케이션이 정상 실행으로 돌아오지 못하는 방향으로 상태가 변한다
A와 B 둘다 교착상태에서 빠져나오는 로직이 같다고 했을때 둘다 같은 방향으로 변하여 또 다시 서로가 교착이 발생하는 현상을 말한다.




## Kotlin에서의 동시성

코루틴을 다른 스레드로 이동시키는 역할은 런타임이 담당한다.

### NonBlocking
Thread는 무겁고 생성하는데에 비용이 크다. 그래서 생성하는데도 제한이 있다.
Thread가 Blocking되면 자원이 심하게 낭비되는 셈이지만 `Kotlin`은 `Suspendable Computations` (중단 가능 연산)을 제공한다
`Suspendable Computations`은 Thread를 Blocking하지 않으면서 실행을 잠시 중단하고 해당 스레드를 다른 연산에 사용한다.

Kotlin은 Channel(채널), Actors(애겉), Mutural exclusions(상호 배제) 와 같은 기본형도 제공해 스레드를 블록하지 않고 동시성 코드를 효과적으로 통신하고 등기화하는 메커니즘을 제공

> 관례적으로 동시에 실행될 함수는 async로 시작하거나 Async로 끝나도록 이름을 짓도록 한다.

### 유연성
#### Channel (채널)
코루틴 간에 데이터를 안전하게 보내고 받는데 사용할 수 있는 Pipe

#### Worker pools(작업자 풀)
많은 스레드에서 연산 집합의 처리를 나눌 수 있는 코루틴 풀

#### Mutexes (뮤텍스)
`Critical Zone` 영역을 정의해 한 번에 하나의 스레드만 실행 할 수 있도로 하는 동기화 메커니즘
`Critical Zone`에 엑세스하려는 코루틴은 이전 코루틴이 `Critical Zone`을 빠져나올 때까지 일시 정지된다.

#### Thread Confinement (스레드 한정)
코루틴의 실행을 제한해서 지정된 스레드에서만 실행하도록 하는 기능

#### Constructor (생성자)
필요에 따라 정보를 생성할 수 있고 새로운 정보가 필요하지 않을 때 일시 중단된 수 있는 데이터 소스

### Kotlin의 기능
#### Suspendable Computations (일시 중단 연산)
해당 스레드를 차단하지 않고 실행을 일시 중지할수 있는 연산
`Suspendable Computations`은 Thread를 Blocking하지 않으면서 실행을 잠시 중단하고 해당 스레드를 다른 연산에 사용한다.
> Suspendable Computations은 다른 일시 중단 함수 또는 코루틴에서만 호출되는 특징이 있다.

#### Suspend Function (일시 중단 함수)
일시 중단 함수는 함수 형식의 일시 중단 연산
suspend 연산자를 이용하여 사용된다

#### Suspend Lambda (람다 일시 중단)
일시중단 람다는 다른 suspend function을 호출함으로 자신의 실행을 중단할 수 있다.

#### Coroutine Dispatcher
코루틴을 시작하거나 재개할 스레드를 결정하기 위해 코루틴 디스패처가 사용된다.
모든 코루틴 디스패처는 CoroutineDispatcher 인터페이스를 구현해야 한다.
- Dispatchers.Default : CPU 사용량이 많은 작업에 사용. Main thread에서 작업하기에는 너무 긴 작업에 최적화
- Dispatchers.IO : 네트워크, 디스크 사용 할때 사용. 파일 읽고, 쓰고, 소켓을 읽고, 쓰고 작업을 멈추는것에 최적화
- Dispatchers.Main : 안드로이드의 경우 UI 스레드를 사용
- Dispatchers.Unconfined : 현재 스레드(코루틴이 호출된 스레드)에서 코루틴을 시작하지만 어떤 스레드에서도 코루틴이 다시 재개될 수 있다. Dispatcher에는 스레드 정책을 사용하지 않는다.

> Dispatchers.Main은 `kotlinx-coroutines-android`를 추가해주어야한다.

Dispatcher와 같께 필요에 따라 Pool 또는 Thread를 정의하는데 사용할 수 있는 빌더
- `newSingleThreadContext()` : 생성되면 필요한 만큼 많은 코루틴을 수행하는데 사용할 수 있다.
- `newFixedThreadPoolContext()` : 스레드 풀 생성

#### Coroutine Builder
##### async
결과가 예상되는 코루틴을 시작하는데 사용
async는 코루틴 내부에서 일어나는 모든 예외를 캡처해서 결과에 넣기 때문에 조심해서 사용해야 한다.  
결과 또는 예외를 초함하는 `Deferred<T>를` 반환

##### launch
결과를 반환하지 않는 코루틴을 시작
자체 혹은 자식 코루틴의 실행을 취소하기 위해 사용하는 `Job`을 반환

##### runBlocking
블로킹 코드를 일시 중지 가능한 코드로 연결하기 위해 존재
main 메소드와 유닛 테스트에서 사용
runBlocking은 코루틴의 실행이 끝날 때까지 현재 스레드를 차단


