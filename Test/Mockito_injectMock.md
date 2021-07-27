## Mockito InjectMock을 interface에 사용하기

``` java
// Service.java
public interface Service {
    void test();
}


// ServiceImpl.java
@Service
public class ServiceImpl extends Service {
    
    private Repository repository;
    
    public ServiceImpl(Repository repository) {
        this.repository = repository;
    }
    
    public void print(int index) {
        int i = repository.findByItem(index);
        System.out.println("i : " + i);
    }
}

// Repository.java
public class Repository {
    
    private List<String> anyList = List.of(1, 2, 3, 4);
    
    public findByItem(int index) {
        return anyList.stream()
            .filter(it -> it == index)
            .findFirst()
            .orElse(0);
    }
}
```

위와 같은 일반적인 상황인 경우 테스트를 할때 Mockking을 해야 하는 경우가 있다.


``` java
@ExtendWith(MockitoExtension.class)
class ServiceTest() {

    @InjectMocks
    private Service service;

    @Mock
    private Repository repository;

    @Test
    void test() {
        when(repository.findByItem(0)).thenReturn(0);
        gostopService.print(0);
    }
}
```
위와 같은 상황에서 service에 InjectMocks를 적용시키고 repository를 모킹하여 0이 반환되는걸 예상하는 테스트 코드이다.

하지만 위는 실행하면 에러가 떨어지게된다.

```
Cannot instantiate @InjectMocks field named 'service'! Cause: the type 'service' is an interface.
```
서비스가 interface라서 구체화를 할 수 없다는 내용의 에러다

여기서 해결하는 방법이 있다.

``` java
    @InjectMocks
    private ServiceImpl service;
}
```

`@InjectMocks`을 적용했던 Service를 ServiceImpl로 선언하면 된다.

하지만 우리는 concrete type을 직접 사용하기엔 interface의 조건을 충족시키기 힘들 수 있다.

interface의 스펙을 지키면서 테스트를 하고 싶은 방법은 아래와 같이 할 수 있다.

``` java
    @InjectMocks
    private Service service = new ServiceImpl();

    @Mock
    private Repository repository;
```
위와같이 한다면 Service에 대한 interface를 테스트 할 수 있는데 `new ServiceImpl()` 부분에서 컴파일 에러가 떨어진다.
`Repository`를 DI 해줘야 하기 때문이다.

``` java
    @Mock
    private Repository repository;
    
    @InjectMocks
    private Service service = new ServiceImpl(repository);
```
이런 형태도 직접 넣을 수가 없기 때문에 에러가 발생하게된다.

그래서 lazy 하게 넣어주기 위해서 `@BeforeEach`를 넣어줘야한다.

``` java
@ExtendWith(MockitoExtension.class)
class ServiceTest() {

    private Service service;

    @Mock
    private Repository repository;

    @BeforeEach
    void setup() {
        service = new ServiceImpl(repository);
    }

    @Test
    void test() {
        when(repository.findByItem(0)).thenReturn(0);
        gostopService.print(0);
    }
}
```
위와 같이 자동으로 DI를 제외하고 Mockkin한 객체를 직접 주입해준다면 Bean을 생성, 읽지 않고도 빠르게 Test가 가능하게 된다.
