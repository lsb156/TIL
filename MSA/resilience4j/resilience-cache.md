# Cache
`resilience4j-cache`는 `javax.cache.Cache`를 wrapping해서 사용한다. `javax.cache.Cache` instance를 만들고 그 instance를 `io.github.resilience4j.cache.Cache`로 wrapping한다.

캐시 추상화는 람다 식의 결과를 캐시 인스턴스 ( JCache )에 넣고 람다 식을 호출하기 전에 캐시에서 이전 캐시 된 결과를 검색
분산 캐시에서 캐시 검색이 실패하면 예외가 처리되고 람다식이 호출

> 일부 동시성 문제가 발생하므로 프로덕션에서 JCache 참조 구현 을 사용하지 않는 것이 좋다.
> Ehcache, Caffeine, Redisson, Hazelcast, Ignite 또는 기타 JCache를 사용할것을 권장

``` java
// Create a CacheContext by wrapping a JCache instance.
javax.cache.Cache<String, String> cacheInstance = Caching
  .getCache("cacheName", String.class, String.class);
Cache<String, String> cacheContext = Cache.of(cacheInstance);

// Decorate your call to BackendService.doSomething()
CheckedFunction1<String, String> cachedFunction = Decorators
    .ofCheckedSupplier(() -> backendService.doSomething())
    .withCache(cacheContext)
    .decorate();
String value = Try.of(() -> cachedFunction.apply("cacheKey")).get();
```

## Consume emitted CacheEvents
``` java
cacheContext.getEventPublisher()
    .onCacheHit(event -> logger.info(...))
    .onCacheMiss(event -> logger.info(...))
    .onError(event -> logger.info(...));
```



## Ehcache example
``` gradle
compile 'org.ehcache:ehcache:3.7.1'
```

``` java
// Configure a cache (once)
this.cacheManager = Caching.getCachingProvider().getCacheManager();
this.cache = Cache.of(cacheManager
    .createCache("booksCache", new MutableConfiguration<>()));

// Get books using a cache
List<Book> books = Cache.decorateSupplier(cache, library::getBooks)
    .apply(BOOKS_CACHE_KEY);
```
