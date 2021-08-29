```
dependencies {
	compile("org.springframework.boot:spring-boot-starter-actuator")
}
```
### 등록
`management.endpoint.shutdown.enabled=true //false`


### health detail 하게 표시
```
management.endpoint.health.show-details=always
```

### custom indicator
`AbstractHealthIndicator`를 상속받아서 만든다.
class 이름을 `XXXHealthIndicator`로 한다면 `/actuator/health` 접속시에 `XXX` 라는 인디게이터가 떠있다.

바로 `HealthIndicator`를 구현할 경우
``` java
@Component
public class MyHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check();
        if (errorCode != 0) {
            return Health.down().withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

    private int check() {
        // perform some specific health check
        return ...
    }

}
```


### CORS Support
```
management:
  endpoints:
    web:
      cors:
        allowed-origins: "https://example.com"
        allowed-methods: "GET,POST"
```



### with spring security
```
@Configuration
public class ActuatorSecurityConfig extends WebSecurityConfigurerAdapter {

    /*
        This spring security configuration does the following

        1. Restrict access to the Shutdown endpoint to the ACTUATOR_ADMIN role.
        2. Allow access to all other actuator endpoints.
        3. Allow access to static resources.
        4. Allow access to the home page (/).
        5. All other requests need to be authenticated.
        5. Enable http basic authentication to make the configuration complete.
           You are free to use any other form of authentication.
     */

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                    .requestMatchers(EndpointRequest.to(ShutdownEndpoint.class))
                        .hasRole("ACTUATOR_ADMIN")
                    .requestMatchers(EndpointRequest.toAnyEndpoint())
                        .permitAll()
                    .requestMatchers(PathRequest.toStaticResources().atCommonLocations())
                        .permitAll()
                    .antMatchers("/")
                        .permitAll()
                    .antMatchers("/**")
                        .authenticated()
                .and()
                .httpBasic();
    }
}
```
#### security filter chain
```
@Configuration(proxyBeanMethods = false)
public class MySecurityConfiguration {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.requestMatcher(EndpointRequest.toAnyEndpoint())
                .authorizeRequests((requests) -> requests.anyRequest().hasRole("ENDPOINT_ADMIN"));
        http.httpBasic();
        return http.build();
    }

}
```


#### actuator security
``` groovy
compile 'org.springframework.boot:spring-boot-starter-security'
```

``` java
@Configuration
@AllArgsConstructor
public class SecuritySecureConfig extends WebSecurityConfigurerAdapter {
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers(UrlConstants.SECURE + "/**").authenticated()
            .anyRequest().permitAll()
            .and()
            .httpBasic()
            .and()
            .csrf().disable();
    }
     
}
```

``` yaml
spring:
  security:
    user:
      name: {{usrename}}
      password: {{password}}

```

> https://reflectoring.io/spring-boot-health-check/
> https://godekdls.github.io/Spring%20Boot/endpoints/
> https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html
