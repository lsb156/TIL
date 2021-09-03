``` java
@Configuration
public class FeignClientConfiguration {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

``` yaml

logging:
  level:
    com.package.project:
      adapter:
        externalapi: DEBUG
```
feign 설정에 `Logger.Level.FULL`로 해놓았어도
properties상에 DEBUG 레벨일때만 출력.
beta, real 환경에서는 INFO로 출력하지 않는다.
