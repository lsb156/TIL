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
