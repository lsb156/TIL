# TimeLimiter
TimeLimiter 인스턴스를 관리 (생성 및 검색)하는 데 사용할 수 있는 인 메모리 를 제공

||Description|Default|
|:--|:--|:-:|
|timeoutDuration|실행 timeout 값|1(sec)|
|cancelRunningFuture|timeout 발생 후 future를 취소할지 결정하는 boolean 값|true|


## Create Retry Configuration
``` java
// Default
TimeLimiterRegistry timeLimiterRegistry = TimeLimiterRegistry.ofDefaults();

// Custom
TimeLimiterConfig config = TimeLimiterConfig.custom()
   .cancelRunningFuture(true)
   .timeoutDuration(Duration.ofMillis(500))
   .build();

// Create a TimeLimiterRegistry with a custom global configuration
TimeLimiterRegistry timeLimiterRegistry = TimeLimiterRegistry.of(config);

// Get or create a TimeLimiter from the registry - 
// TimeLimiter will be backed by the default config
TimeLimiter timeLimiterWithDefaultConfig = registry.timeLimiter("name1");

// Get or create a TimeLimiter from the registry, 
// use a custom configuration when creating the TimeLimiter
TimeLimiterConfig config = TimeLimiterConfig.custom()
   .cancelRunningFuture(false)
   .timeoutDuration(Duration.ofMillis(1000))
   .build();

TimeLimiter timeLimiterWithCustomConfig = registry.timeLimiter("name2", config);
```
