# Resilience Bulkhead Example

## Common BulkHead

### Create Bulkhead
``` java
BulkheadRegistry bulkheadRegistry = BulkheadRegistry.ofDefaults();

ThreadPoolBulkheadRegistry threadPoolBulkheadRegistry = 
    ThreadPoolBulkheadRegistry.ofDefaults();

// Create a custom configuration for a Bulkhead
BulkheadConfig config = BulkheadConfig.custom()
    .maxConcurrentCalls(150)
    .maxWaitDuration(Duration.ofMillis(500))
    .build();

// Create a BulkheadRegistry with a custom global configuration
BulkheadRegistry registry = BulkheadRegistry.of(config);

// Get or create a Bulkhead from the registry - 
// bulkhead will be backed by the default config
Bulkhead bulkheadWithDefaultConfig = registry.bulkhead("name1");

// Get or create a Bulkhead from the registry, 
// use a custom configuration when creating the bulkhead
Bulkhead bulkheadWithCustomConfig = registry.bulkhead("name2", custom);
```


## ThreadPool Bulkhead

``` java
ThreadPoolBulkheadConfig config = ThreadPoolBulkheadConfig.custom()
  .maxThreadPoolSize(10)
  .coreThreadPoolSize(2)
  .queueCapacity(20)
  .build();
        
// Create a BulkheadRegistry with a custom global configuration
ThreadPoolBulkheadRegistry registry = ThreadPoolBulkheadRegistry.of(config);

// Get or create a ThreadPoolBulkhead from the registry - 
// bulkhead will be backed by the default config
ThreadPoolBulkhead bulkheadWithDefaultConfig = registry.bulkhead("name1");

// Get or create a Bulkhead from the registry, 
// use a custom configuration when creating the bulkhead
ThreadPoolBulkheadConfig custom = BulkheadConfig.custom()
  .maxThreadPoolSize(5)
  .build();

ThreadPoolBulkhead bulkheadWithCustomConfig = registry.bulkhead("name2", custom);
```



## Bulkhead Event
### Consume emitted Registry Events
``` java
BulkheadRegistry registry = BulkheadRegistry.ofDefaults();
registry.getEventPublisher()
  .onEntryAdded(entryAddedEvent -> {
    Bulkhead addedBulkhead = entryAddedEvent.getAddedEntry();
    LOG.info("Bulkhead {} added", addedBulkhead.getName());
  })
  .onEntryRemoved(entryRemovedEvent -> {
    Bulkhead removedBulkhead = entryRemovedEvent.getRemovedEntry();
    LOG.info("Bulkhead {} removed", removedBulkhead.getName());
  });
```


### Consume emitted Bulkhead Event
``` java
bulkhead.getEventPublisher()
    .onCallPermitted(event -> logger.info(...))
    .onCallRejected(event -> logger.info(...))
    .onCallFinished(event -> logger.info(...));
```
