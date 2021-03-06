# Retry
Retry 인스턴스를 관리(생성 및 검색) 하는 데 사용할 수 있는 인 메모리 환경을 제공

||Description|Default|
|:--|:--|:-:|
|maxAttempts|최대 시도 횟수 (초기 호출을 첫 번째 시도로 포함)|3|
|waitDuration|재시도 사이의 고정 된 대기 시간|500(ms)|
|intervalFunction|장애 발생 후 대기 간격을 수정하는 기능.<br/> 기본적으로 대기 시간은 일정하게 유지|`numOfAttempts`-> `waitDuration`|
|intervalBiFunction|시도 횟수 및 결과 또는 예외에 따라 실패 후 대기 간격을 수정하는 기능 <br/> `intervalFunction`과 함께 사용하면 IllegalStateException이 발생|(numOfAttempts, Either <throwable, result)-> waitDuration|
|retryOnResultPredicate|결과를 재 시도여부를 판단하는 Predicate를 구성<br/> 결과를 재 시도해야하는 경우 true를 반환|result -> false|
|retryOnExceptionPredicate|예외를 재 시도여부를 판단하는 Predicate를 구성<br/>예외를 재 시도해야하는 경우 true를 반환|throwable -> true|
|retryExceptions|실패로 기록되어 재 시도되는 Throwable 클래스 목록을 구성<br/>CheckedExceptions를 사용하는 경우 CheckedSupplier를 사용해야함|empty|
|ignoreExceptions|무시되어 재 시도되지 않는 Throwable 클래스 목록을 구성|empty|
|failAfterMaxRetries|재 시도가 구성된 `maxAttempts`에 도달하고 결과가 여전히 `retryOnResultPredicate`를 전달하지 않을 때 `MaxRetriesExceededException` 발생 여부 값|false|
