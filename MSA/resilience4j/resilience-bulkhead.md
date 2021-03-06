# Bulkhead
Bulkhead는 동시 실행수 제한이 가능한 라이브러리
Resilience4j는 동시 실행 수를 제한하는 데 사용할 수있는 벌크 헤드 패턴의 두 가지 구현을 제공

- 세마포어 사용 : `SemaphoreBulkhead`
- 바운드 대기열 고정 스레드 풀을 사용 : `FixedThreadPoolBulkhead`

## Common BulkHead
### Create Base Configuration
||Description|Default|
|:--|:--|:-:|
|maxConcurrentCalls|벌크 헤드에서 허용되는 최대 병렬 실행 수|25|
|maxWaitDuration|가득찬 세마포어에를 대기하는 스레드의 sleep 시간|0|

## Create ThreadPoolBulkHead Configuration
ThreadPool 기반의 Bulkhead 만들기
||Description|Default|
|:--|:--|:-:|
|maxThreadPoolSize|최대 스레드 풀 크기|Runtime.getRuntime().availableProcessors()|
|coreThreadPoolSize|코어 스레드 풀 크기|Runtime.getRuntime().availableProcessors() - 1|
|queueCapacity|Queue size|100|
|keepAliveDuration|스레드 수가 코어보다 많으면 초과 유휴 스레드가 종료되기 전에 새 작업을 기다리는 최대 시간|20(ms)|
