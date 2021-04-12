- spring-boot 2.4.2
- spring-cloud 2020.0.1

``` gradle
implementation 'org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j'
implementation 'io.github.resilience4j:resilience4j-spring-boot2'
implementation 'io.github.resilience4j:resilience4j-micrometer'
```

> 옵션 설정 방법
> - https://resilience4j.readme.io/docs/getting-started-3#configuration
> - https://resilience4j.readme.io/docs/circuitbreaker

``` yaml
resilience4j.circuitbreaker:
  configs:
    default:
      registerHealthIndicator: true
      slidingWindowSize: 10
      minimumNumberOfCalls: 5
      permittedNumberOfCallsInHalfOpenState: 3
      automaticTransitionFromOpenToHalfOpenEnabled: true
      waitDurationInOpenState: 5s
      failureRateThreshold: 50
      eventConsumerBufferSize: 10
resilience4j.timelimiter:
  configs:
    default:
      timeoutDuration: 3s
      cancelRunningFuture: true #timeout 발생 시 중간에 강제 종료를 원할때 true

```


``` java
class TestService {

    @CircuitBreaker(name = "ProductRemoteService#findProduct", fallbackMethod = "throwFallback")
    public Product findProduct(Long productId) {
        final String url = UriComponentsBuilder.fromHttpUrl(productApi + "/spring-demo/mvc/v1.0/products/" + productId)
                .toUriString();
        return restTemplate.getForObject(url, Product.class);
    }
    
    public Product throwFallback(Long productId, Throwable e) {
        throw new RuntimeException(e);
    }
}
```

## FeignConfig로 설정

```java
@FeignClient(name = "spring-demo-product", configuration = ProductApi.FeignConfiguration.class)
public interface ProductApi {

	@GetMapping("/spring-demo/mvc/v1.0/products/{productId}")
	Product findProduct(@PathVariable Long productId);

	@GetMapping("/spring-demo/mvc/v1.0/products/search")
	List<Product> findProductsByProductIds(@RequestParam String searchKey,
										   @RequestParam List<String> searchValue);
	
	/**
	 * 다양한 옵션 설정
	 */
	@Configuration
	class FeignConfiguration {

		/**
		 * connection 에 관련된 옵션
		 *
		 * @return
		 */
		@Bean
		public Request.Options options() {
			return new Request.Options(3, TimeUnit.SECONDS,
					300, TimeUnit.MILLISECONDS, true);
		}

		@Bean
		public Retryer retryPolicies() {
			return new Retryer.Default(100, 200, 5);
		}

		/**
		 * 에러 응답에 대한 에러 처리
		 *
		 * @return
		 */
		@Bean
		public feign.codec.ErrorDecoder reactiveStatusHandler() {
			return new ErrorDecoder();
		}


		@Bean
		public TestFallbackFactory testFallbackFactory() {
			return new TestFallbackFactory();
		}


		@Bean
		public Resilience4JCircuitBreakerFactory circuitBreakerFactory(MeterRegistry meterRegistry) {
			Resilience4JCircuitBreakerFactory circuitBreakerFactory = new Resilience4JCircuitBreakerFactory();

			circuitBreakerFactory.configureDefault(id -> new Resilience4JConfigBuilder(id)
					.circuitBreakerConfig(CircuitBreakerConfig.ofDefaults())
					.timeLimiterConfig(TimeLimiterConfig.custom()
							.timeoutDuration(Duration.ofSeconds(5)).build()).build());

			CircuitBreakerRegistry circuitBreakerRegistry = CircuitBreakerRegistry.ofDefaults();
			circuitBreakerFactory.configureCircuitBreakerRegistry(circuitBreakerRegistry);

			//prometheus metric 정보를 생성하기 위해..
			TaggedCircuitBreakerMetrics
					.ofCircuitBreakerRegistry(circuitBreakerRegistry)
					.bindTo(meterRegistry);

			return circuitBreakerFactory;
		}

		class ErrorDecoder implements feign.codec.ErrorDecoder {

			@Override
			public Exception decode(String methodKey, Response response) {

				if (HttpStatus.valueOf(response.status()).is5xxServerError()) {
					return new RetryableException(response.status(), format("%s 요청이 성공하지 못했습니다. Retry 합니다. - status: %s, headers: %s", methodKey, response.headers()), response.request().httpMethod(), null, response.request());
				}
				return new IllegalStateException(format("%s 요청이 성공하지 못했습니다. - status: %s, headers: %s", methodKey, response.status(), response.headers()));
			}
		}


		class TestFallbackFactory implements FallbackFactory<ProductMvcApi> {

			@Override
			public ProductMvcApi create(Throwable throwable) {
				return new ProductMvcApi() {
					@Override
					public Product findProduct(Long productId) {
						throw new RuntimeException("findProduct-fallback", throwable);
					}

					@Override
					public List<Product> findProductsByProductIds(String searchKey, List<String> searchValue) {
						throw new RuntimeException("findProductsByProductIds-fallback", throwable);
					}
				};
			}
		}
	}
}
```


## grafana dashboard
- https://github.com/resilience4j/resilience4j/blob/master/grafana_dashboard.json