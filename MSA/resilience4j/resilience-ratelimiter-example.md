
## Create and configure a RateLimiter
``` java
// 10 req / ms
RateLimiterConfig config = RateLimiterConfig.custom()
  .limitRefreshPeriod(Duration.ofMillis(1))
  .limitForPeriod(10)
  .timeoutDuration(Duration.ofMillis(25))
  .build();

// Create registry
RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.of(config);

// Use registry
RateLimiter rateLimiterWithDefaultConfig = rateLimiterRegistry
  .rateLimiter("name1");

RateLimiter rateLimiterWithCustomConfig = rateLimiterRegistry
  .rateLimiter("name2", config);
```

## RateLimiter Event
### Consume emitted Registry Events
``` java
RateLimiterRegistry registry = RateLimiterRegistry.ofDefaults();
registry.getEventPublisher()
  .onEntryAdded(entryAddedEvent -> {
    RateLimiter addedRateLimiter = entryAddedEvent.getAddedEntry();
    LOG.info("RateLimiter {} added", addedRateLimiter.getName());
  })
  .onEntryRemoved(entryRemovedEvent -> {
    RateLimiter removedRateLimiter = entryRemovedEvent.getRemovedEntry();
    LOG.info("RateLimiter {} removed", removedRateLimiter.getName());
  });
```


### Consume emitted RateLimiter Event
``` java
rateLimiter.getEventPublisher()
    .onSuccess(event -> logger.info(...))
    .onFailure(event -> logger.info(...));
```
