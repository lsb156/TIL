# Spring Cloud Gateway CORS 설정


``` yaml
spring:
  cloud:
    gateway:
      globalcors:
        corsConfigurations:
          '[/**]':
            allowedOrigins: 
              - "http://localhost:3000"
            allowedHeaders:
              - x-requested-with
              - authorization
              - content-type
              - credential
              - X-AUTH-TOKEN
              - X-CSRF-TOKEN
            allowedMethods:
              - POST
              - GET
              - PUT
              - OPTIONS
              - DELETE
```
Swagger에서 OPTIONS를 이용하여 api 호출 이전에 한번 호출함으로 Methods에 `OPTIONS`를 등록해주어야 한다.

CORS 설정은 `config-refresh`로 적용되지 않고 재부팅을 이용해서 적용이된다.`
