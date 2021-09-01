``` java
public class MsaFeignConfig {

    @Bean
    public RequestInterceptor requestInterceptor() {
        return new ForwardHostRequestInterceptor();
    }

    public class ForwardHostRequestInterceptor implements RequestInterceptor {
        private static final String HOST_HEADER = "Host";
        @Override
        public void apply(RequestTemplate requestTemplate) {
            ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            if (requestAttributes == null) {
                return;
            }
            HttpServletRequest request = requestAttributes.getRequest();
            String host = request.getHeader(X_FORWARDED_HOST);
            String clientIp = request.getHeader(X_FORWARDED_FOR);
            if (host == null) {
                host = request.getHeader(HOST_HEADER);
            }

            if (clientIp == null) {
                clientIp = request.getRemoteAddr();
            }

            requestTemplate.header(X_FORWARDED_HOST, host);
            requestTemplate.header(X_FORWARDED_FOR, clientIp);
        }
    }
}

```

``` java
@FeignClient(name = "${feign.test}", configuration = MsaFeignConfig.class)
public interface TestClient {
    ...
}
```
