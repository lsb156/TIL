# RateLimiter
일정시간동안 요청의 수를 제한하는데 사용되는 라이브러리

## Create RateLimiter Configuration
||Description|Default|
|:--|:--|:-:|
|timeoutDuration|스레드가 권한을 기다리는 기본 대기 시간|5(sec)|
|limitRefreshPeriod|limit count(limitForPeriod 값을 사용)가 refresh되는 period duration|500(ns)|
|limitForPeriod|하나의 period(limitRefreshPeriod 값을 사용)마다 허용하는 limit count|50|
