# Resilience4J

> Resilience4j is a lightweight, easy-to-use fault tolerance library inspired by Netflix Hystrix.

`Hystrix`로부터 영감을 받아 만들어진 `fault tolerance library`이며 Maintenance Mode 로 들어간 `Hystrix`이후 주목을 받는 대체제이다.

> `Hystrix` Github 에서도 `Resilience4j`를 사용하기를 권장하고있다.

Resilience4j에는 다음과 같은 구현체들이 존재한다.
- Circuit Breaker
  - [resilience-circuitbreaker](resilience-circuitbreaker.md)
  - [resilience-circuitbreaker-example](resilience-circuitbreaker-example.md)
- Rate Limier
  - [resilience-ratelimiter](resilience-ratelimiter.md)
  - [resilience-ratelimiter-example](resilience-ratelimiter-example.md)
- Bulkhead
- Time Limiter
- Retry
- Cache
