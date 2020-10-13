


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


