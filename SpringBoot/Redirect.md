
## RedirectView
``` java
@Controller
@RequestMapping("/")
public class RedirectController {
    
    @GetMapping("/redirectWithRedirectView")
    public RedirectView redirectWithUsingRedirectView(
      RedirectAttributes attributes) {
        attributes.addFlashAttribute("flashAttribute", "redirectWithRedirectView");
        attributes.addAttribute("attribute", "redirectWithRedirectView");
        return new RedirectView("redirectedUrl");
    }
}

```
`RedirectView`는 `HttpServletResponse.sendRedirect()`를 트리거
model에는 문자열로 변환이 가능한 객체만 기능
``` bash
# request
curl -i http://localhost:8080/spring-rest/redirectWithRedirectView

# response
HTTP/1.1 302 Found
Server: Apache-Coyote/1.1
Location: 
  http://localhost:8080/spring-rest/redirectedUrl?attribute=redirectWithRedirectView

```
RedirectView는 Spring API에 의존적
Controller를 설계할때 항상 이 요청은 redirect에 묶여 있습니다. (return값이 RedirectView)


## Prefix
spring에서는 view를 응답할때 view응답 대신 `redirect:/` 를 통하여 리다이렉트가 가능
이 작업은 `UrlBasedViewResolver`에서 리다이렉션 요구가 발생하는것을 인지.
`redirect:/redirectUrl`에 있는 url은 현재 서블릿 컨텍스트에 상대경로로 수행

``` java
@Controller
@RequestMapping("/")
public class RedirectController {
    
    @GetMapping("/redirectWithRedirectPrefix")
    public ModelAndView redirectWithUsingRedirectPrefix(ModelMap model) {
        model.addAttribute("attribute", "redirectWithRedirectPrefix");
        return new ModelAndView("redirect:/redirectedUrl", model);
    }
}
```

``` bash
# request
curl -i http://localhost:8080/spring-rest/redirectWithRedirectPrefix

# response
HTTP/1.1 302 Found
Server: Apache-Coyote/1.1
Location: 
  http://localhost:8080/spring-rest/redirectedUrl?attribute=redirectWithRedirectPrefix

```

## request Attribute를 통해 전달
``` java
@RequestMapping(value="/forwardWithParams", method = RequestMethod.GET)
public ModelAndView forwardWithParams(HttpServletRequest request) {
    request.setAttribute("param1", "one");
    request.setAttribute("param2", "two");
    return new ModelAndView("forward:/forwardedWithParams");
}
```


## Post -> Post Redirection
301, 302 Redirection은 GET으로 변경이 될 상황이 있어
POST에서 GET으로 변경되는 것을 허용하지 않는 경우는 해당 307(임시 리디렉션) 및 308(영구 리디렉션) 상태 코드를 정의하여 응답한다.
``` java
@PostMapping("/redirectPostToPost")
public ModelAndView redirectPostToPost(HttpServletRequest request) {
    request.setAttribute(View.RESPONSE_STATUS_ATTRIBUTE, HttpStatus.TEMPORARY_REDIRECT);
    return new ModelAndView("redirect:/redirectedPostToPost");
}

@PostMapping("/redirectedPostToPost")
public ModelAndView redirectedPostToPost() {
    return new ModelAndView("redirection");
}
```
``` bash
# request
curl -L --verbose -X POST http://localhost:8080/spring-rest/redirectPostToPost

# response
> POST /redirectedPostToPost HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.49.0
> Accept: */*
> 
< HTTP/1.1 200 
< Content-Type: application/json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Tue, 08 Aug 2017 07:33:00 GMT

{"id":1,"content":"redirect completed"}
```
