consul health check 설정 변경
``` yaml
spring:
  cloud:
    consul:
      discovery:
        health-check-interval: 1s
        health-check-timeout: 5s
        health-check-path: /monitor/health
```
