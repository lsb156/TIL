## openfeign 동적 URI

Feign은 Annotation이 없는 URI 객채에 대해서는 기본 요청 주소로 받아들인다.
``` java
@FeignClient(name = "dummy-name", url = "https://this-is-a-placeholder.com")
public interface MyClient {
    @PostMapping(path = "/create")
    UserDto createUser(URI baseUrl, @RequestBody UserDto userDto);
}
```
