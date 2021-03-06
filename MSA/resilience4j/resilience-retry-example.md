# Retry

## Create Retry Configuration
``` java
RetryConfig config = RetryConfig.custom()
  .maxAttempts(2)
  .waitDuration(Duration.ofMillis(1000))
  .retryOnResult(response -> response.getStatus() == 500)
  .retryOnException(e -> e instanceof WebServiceException)
  .retryExceptions(IOException.class, TimeoutException.class)
  .ignoreExceptions(BusinessException.class, OtherBusinessException.class)
  .failAfterMaxAttempts(true)
  .build();

// Create a RetryRegistry with a custom global configuration
RetryRegistry registry = RetryRegistry.of(config);

// Get or create a Retry from the registry - 
// Retry will be backed by the default config
Retry retryWithDefaultConfig = registry.retry("name1");

// Get or create a Retry from the registry, 
// use a custom configuration when creating the retry
RetryConfig custom = RetryConfig.custom()
    .waitDuration(Duration.ofMillis(100))
    .build();

Retry retryWithCustomConfig = registry.retry("name2", custom);
```

## RateLimiter Event
### Consume emitted Registry Events
``` java
RetryRegistry registry = RetryRegistry.ofDefaults();
registry.getEventPublisher()
  .onEntryAdded(entryAddedEvent -> {
    Retry addedRetry = entryAddedEvent.getAddedEntry();
    LOG.info("Retry {} added", addedRetry.getName());
  })
  .onEntryRemoved(entryRemovedEvent -> {
    Retry removedRetry = entryRemovedEvent.getRemovedEntry();
    LOG.info("Retry {} removed", removedRetry.getName());
  });
```

