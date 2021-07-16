# HATEOAS

## HATEOAS References

기본적으로 2가지의 형태로 제공
### WebLink(RFC5988)
https://tools.ietf.org/html/rfc5988
- Target URI:
  - 전이, 요청을 할 수 있는 link.
  - `href`라고 표현
- Link Relation Type:
  - target resource의 URI와 현재 URI가 어떻게 연결되어있는지 관계에 대해 정의
  - `rel`라고 표현
- Attribute for target URI:
  - target link를 부가 설명할 수 있는 요소
  - title, media, type 및 정의에 따라 사용

### HAL (Hypermedia API Language)
https://tools.ietf.org/html/draft-kelly-json-hal-03
- HAL은 json또는 xml의 content에 link를 담는 방식

아래 2개의 contentType 사용하여 표현
- `application/hal+xml`, `application/hal+json`
```yaml
{
  "_links": {
    "self": {
      "href": "/boards"
    }
  },
  "_embeded": {
    "boards": [ {
      "idx": 1,
      "title": "1 title",
      "contents": "1 contents"
        "_links": {
          "self": {
            "href": "/boards/1"
          }
        }
    },{
      "idx": 2,
      "title": "2 title",
      "contents": "2 contents"
        "_links": {
          "self": {
            "href": "/boards/2"
          }
        }
    } ]
  }
}
```

- `_link` : 호출한 resource와 연관있는 link들을 collection의 형식으로 표현
- `_embedded` : 해당 resource가 포함하고 있는 요소를 표현
- `self` : 각 Resource에 대해서 본인의 Resource에 대한 link는 반드시 포함



## 프로젝트에 적용
https://docs.spring.io/spring-hateoas/docs/current/reference/html

``` groovy
// build.gradle
dependencies {
    /** Hateoas */
    implementation 'org.springframework.boot:spring-boot-starter-hateoas'
}
```

``` java 
// PaymentController.java
public class PaymentController {

    private final PaymentService paymentService;

    @GetMapping(value = "/payment", produces = {MediaType.ALL_VALUE, MediaType.APPLICATION_JSON_VALUE})
    public ResponseEntity<Response> itemPaymentInformation(Request request) {
        PaymentInfo paymentInfo = paymentService.retrievePayment(request.getCode());
        Response response = Response.of(paymentInfo);
        return ResponseEntity.ok(response);
    }

    @GetMapping(value = "/v1.0/payment", produces = MediaTypes.HAL_JSON_VALUE)
    public ResponseEntity<Object> itemPaymentInformationForHalJson(Request request) {
        ResponseEntity<Response> response = itemPaymentInformation(request);
        ItemPaymentInfoDto.Response body = response.getBody();
        return ResponseEntity.ok(
                HalModelBuilder.emptyHalModel()
                        .entity(ResponseForHal.of(body.getHancoin()))
                        .embed(body.getAnyList(), LinkRelation.of("list"))
                        .embed(body.getAnotherList(), LinkRelation.of("anotherList"))
                        .link(linkTo(methodOn(getClass()).itemPaymentInformation(null)).withSelfRel())
                        .link(linkTo(getClass()).withRel("profile").withHref("http://domain/v1.0/docs/index.html"))
                        .build());
    }

    @Getter
    @AllArgsConstructor(staticName = "of")
    public static class ItemPaymentInformationForHal {
        private long hancoin;
    }
}
```

`application/json`, `application/hal+json` 두 요청을 다 받기 위해서 아래와 같이 두개의 메소드를 설정하여준다 .
- `itemPaymentInformation` : `produces = {MediaType.ALL_VALUE, MediaType.APPLICATION_JSON_VALUE}`
- `itemPaymentInformationForHalJson` : `produces = MediaTypes.HAL_JSON_VALUE`
1. `itemPaymentInformationForHalJson`로  `application/hal+json` 요청이 들어오면
2.  `application/json`요청이 들어오는 `itemPaymentInformation` 메소드를 호출하여 데이터를 받고
3.  안에 body를 꺼내서 우리가 원하는 형태의 데이터를 만들어준다.
  - entity : Object 내부의 field를 응답 `root`에 정의한다.
  - embed : Collection 형태를 `_embedded` 내부에 정의해준다.
  - link : `_links`에 내용을 채워넣어준다.
    - withRel : link에 표시될 key 이름
    - withHref : link에 표시될 url 값


### Assembler를 이용한 변환
`RepresentationModelAssembler`를 상속받아 Object인 상황과 Collection 형태의 상황을 각각 정의하여 사용이 가능하다.
``` java
@Component
public class EmployeeModelAssembler implements RepresentationModelAssembler<Employee, EmployeeDTO> {
    @Override
    public EmployeeDTO toModel(Employee employee) {
        ModelMapper modelMapper = new ModelMapper();
        EmployeeDTO employeeDto = modelMapper.map(employee, EmployeeDTO.class);
        Link selfLink = linkTo(methodOn(EmployeeController.class).getEmployeeById(employee.getId())).withSelfRel();
        employeeDto.add(selfLink);
        return employeeDto;
    }
    @Override
    public CollectionModel<EmployeeDTO> toCollectionModel(Iterable<? extends Employee> employeesList) {
       ModelMapper modelMapper = new ModelMapper();
       List<EmployeeDTO> employeeDTOS = new ArrayList<>();
       for (Employee employee : employeesList){
           EmployeeDTO employeeDto = modelMapper.map(employee, EmployeeDTO.class);
           employeeDto.add(linkTo(methodOn(EmployeeController.class)
                                .getEmployeeById(employeeDto.getId())).withSelfRel());
           employeeDTOS.add(employeeDto);
        }
        return new CollectionModel<>(employeeDTOS);
    }
}
 
```

### HAL 규칙
https://leedo1982.github.io/wiki/HAL/

### Affordances
더 많은 내용을 담기 위한 ..
https://github.com/spring-projects/spring-hateoas-examples/tree/main/affordances


### Get link에 Parameter 정보 넣기
#### Request를 Object로 받았을때``` java
// 호출
@GetMapping(value = "/v1.0/payment")
public ResponseEntity<Object> test(@Valid RequestDto request) {}
```

``` json
{
    "_links": {
        "self": {
            "href": "http://localhost:8080/"
        }
    }
}
```


#### @RequestParam으로 받았을때
``` java
// 호출 
@GetMapping(value = "/v1.0/payment")
public ResponseEntity<Object> test(@RequestParam String email,
                                   @RequestParam String name) {}
```

``` json
{
    "_links": {
        "self": {
            "href": "http://localhost:8080?email=llsb156@gmail.com&name=Leesangbae"
        }
    }
}
```

#### @RequestParam Annotation 제거했을 경우
``` java
// 호출 
@GetMapping(value = "/v1.0/payment")
public ResponseEntity<Object> test(String email,
                                   String name) {}
```
``` json
{
    "_links": {
        "self": {
            "href": "http://localhost:8080/"
        }
    }
}
```

### X-Forwarded-* 헤더들 위임하기
Spring-Boot 2.1 / Spring 5.1부터 X-Forwarded-*를 처리하는 책임을 Spring HATEOAS 에서 Spring MVC로 이전하여 동작을 안했음

https://github.com/spring-projects/spring-framework/issues/21209

해당 Bean 생성하여 해결
``` java
	@Bean
	FilterRegistrationBean<ForwardedHeaderFilter> forwardedHeaderFilter() {
		FilterRegistrationBean<ForwardedHeaderFilter> bean = new FilterRegistrationBean<>();
		bean.setFilter(new ForwardedHeaderFilter());
		return bean;
	}
```
