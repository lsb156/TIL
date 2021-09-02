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
