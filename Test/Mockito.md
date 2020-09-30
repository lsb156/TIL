# Mockito

Mockito는 모의 객체 생성, 검증, 스텁을 지원하는 프레임워크이다.

`Mock`이란 진자 객체처럼 동작하지만 객체의 행동을 관리하는 객체이며
`Mockito`란 그 Mock을 쉽게 만들고 관리하고 검증할 수 있는 방법을 제공
<!--[TOC level=5]: # "## Table of Contents"-->

## Table of Contents
- [기본 사용법](#기본-사용법)
- [행위 검증](#행위-검증)
- [인자 캡쳐](#인자-캡쳐)


``` xml
<dependencies>
  <dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>2.26.0</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```
``` gradle
dependencies {
    testImplementation('org.mockito:mockito-core:2.26.0')
}
```
Spring Boot에서는 `springboot-test-starter`에 기본적으로 등록이 되어있어 별도로 의존성 추가를 하지 않아도 된다.

## 기본 사용법
``` java
@ExtendWith(MockitoExtension.class)
public MockitoTest{
    
    @Mock
    TestService testService;
    
    @Mock
    TestRepositoyr testRepository;
    
    @Test
    void testMethod(@Mock MemberService memberService) {
        Member member = new Member();
        member.setName("name");
        member.setId(1L);
        // Normal Styel
        when(memberService.findById(1L)).thenReturn(member);
        // BDD Style
        given(memberService.findById(1L)).willReturn(member);

        Member savedMember = memberService.findById(1L);
        assertEquals(1L, savedMember.getId());
        assertEquals("name", savedMember.getName());
        
        // 에러가 나도록 처리
        doThrow(new MemberNotFoundException()).when(memberService).findById(2L);
        
        assertThrow(MemberNotFoundException.class, () -> {
            memberService.findById(2L);
        });
        
        // 지속적인 호출때 값이 다르도록 Stubing 한다.
        when(memberService.findById(1L))
                .thenReturn(member)
                .thenReturn(empty())
                .thenThrow(new MemberNotFoundException());
    }
    
}
```
`@Mock` Annotation이 붙어있는 객체를 Mocking 하려면 `@ExtendWith(MockitoExtension.class)`를 class에 선언해줘야한다.

name, id 값을 가지고 있는 Member를 리턴받기위해 미리 작업을 해 놓는것을 `Stub` 이라고 한다.

``` java
// GameNumGen.java
public interface GameNumGen {
    String generate(GameLevel level);
}

// GameLevel.java
public enum GameLevel {
    EASY, NORMAL, HARD
}

// GameGenMockTest.java
public class GameGenMockTest {

    @Test
    void mockStubTest() {
        // 모의 객체 생성
        GameNumGen genMock = mock(GameNumGen.class);
        // 스텁 설정
        given(genMock.generate(GameLevel.EASY)).willReturn("123");
        // 스텁 설정에 매칭되는 메서드 실행
        String num = genMock.generate(GameLevel.EASY);
        assertEquals("123", num);
    }

    @Test
    void anyMatchTest() {
        GameNumGen genMock = mock(GameNumGen.class);
        // 이전에는 EASY만 "123"을 리턴하게 하였지만 any()를 사용하면 어떤 인자값이든 "456"을 리턴한다
        given(genMock.generate(any())).willReturn("456");

        String num = genMock.generate(GameLevel.EASY);
        assertEquals("456", num);

        String num2 = genMock.generate(GameLevel.NORMAL);
        assertEquals("456", num2);
    }

    @Test
    void mockThrowTest() {
        GameNumGen genMock = mock(GameNumGen.class);
        given(genMock.generate(null)).willThrow(new IllegalArgumentException());

        assertThrows(
                IllegalArgumentException.class,
                () -> genMock.generate(null));
    }

    @Test
    void voidMethodWillThrowTest() {
        List<String> mockList = mock(List.class);
        // 반환값이 void인 경우 Exception을 발생시키기위해 willThrow부터 정의한다.
        willThrow(UnsupportedOperationException.class)
                .given(mockList)
                .clear();

        assertThrows(
                UnsupportedOperationException.class,
                () -> mockList.clear()
        );
    }
}
```


**스텁을 설정할 인자가 두 개 이상인 경우 주의할점**
ArgumentMatchers의 anyInt나 any()등의 메서드는 내부적으로 파라메의 일치 여부를 판단하기 위해 ArgumentMatcher를 등록한다.
Mockito는 한 인자라도 ArgumentMatcher를 사용해서 설정한 경우 모든 인자를 ArgumentMatcher를 이용하여 설정하도록 하고있다.
따라서 다음 코드는 Exception을 발생한다.
``` java
@Test
void mixAnyAndEq() {
    List<String> mockList = mock(List.class);

    given(mockList.set(anyInt(), "123")).willReturn("456");

    String old = mockList.set(5, "123");
    assertEquals("456", old);
}
```
두개의 인자값중 하나는 ArgumentMatcher, 하나는 하드코딩으로 들어가있어서인데 이를 다음과 같이 변경해주어야 한다.
``` java
  given(mockList.set(anyInt(), eq("123"))).willReturn("456");
```
### ArgumentMatcher
ArgumentMatcher는 다음과 같은 메서드를 제공한다

|method|설명|
|:--|:--|
|anyInt()<br>anyShort()<br>anyLong()<br>anyByte()<br>anyChar()<br>anyDouble()<br>anyFloat()<br>anyBoolean()|기본 데이터 타입에 대한 임의 값 일치|
|anyString()|문자열에 대한 임의 값 일치|
|any()|임의 타입에 대한 일치|
|anyList()<br>anySet()<br>anyMap()<br>anyCollection()|임의 콜렉션에 대한 일치|
|matchers(String)<br>matchers(Pattern)|정규표현식을 이용한 String 값 일치 여부|
|eq()|특정 값과 일치 여부|

### Verify
``` java
@ExtendWith(MockitoExtension.class)
public MockitoTest{

    @Mock
    TestService testService;
    
    @Mock
    TestRepositoyr testRepository;
    
    @Test
    void testMethod(@Mock MemberService memberService) {
        Member member = new Member();
        member.setName("name");
        member.setId(1L);
    
        Member member2 = new Member();
        member2.setId(2L);
        
        given(memberService.findById(1L)).willReturn(member);

        Member savedMember = memberService.findById(1L);
        assertEquals(1L, savedMember.getId());
        assertEquals("name", savedMember.getName());
             
        // 정확히 memberService에 member를 이용해 1회 호출되어야 한다.
        verify(memberService, times(1)).findById(member);
        verify(memberService, times(1)).findById(member2);
        verify(memberService, never()).remove(any());

        // 정확히 같은 순서대로 호출이 되어야 한다.
        InOrder inOrder = inOrder(memberService);
        inOrder.verify(memberService).findById(member);
        inOrder.verify(memberService).findById(member2);
    
        // 이제 아무것도 호출이 되면 안된다.    
        verifyNoMoreInteractions(memberService);
        
        // BDD Style
        then(memberService).should(times(1)).notify(study);
        then(memberService).shouldHaveNoMoreInteractions();

    }
}
```

## 행위 검증
다음 코드는 모의 객체의 특정 메서드가 불렸는지 검증하는 코드이다.
``` java
public class GameTest {
    @Test
    void init() {
        GameNumGen genMock = mock(GameNumGen.class);
        Game game = new Game(genMock);;
        game.init(GameLevel.EASY);

		// GameLevel.EASY인자값으로 호출되었는지 확인
        then(genMock).should(only()).generate(GameLevel.EASY);

		// 정확한게 아니라 메서드 호출 유무만 판단할때        
        then(genMock).should().generate(any());
        // 한번만 호출된 것을 검증할때
        then(genMock).should(only()).generate(any());
        
    }
}
```
`BDDMockito.then()`은 메서드 호출 여부를 검증할 모의 객체를 전달받는다.
should() 메서드에서 다음에 실제로 불려야 할 메서드를 지정한다.

메서드 호출 횟수를 검증하기 위해 Mockito에서 제공하는 메소드
|Methos|설명|
|:-|:-|
|only()|한번만 호출|
|times(int)|지정한 횟수만큼 호출|
|never()|호출하지 않음|
|atLeast(int)|적어도 지정한 횟수만큼 호출|
|atLeastOnce()|atLeast(1)과 동일|
|atMost(int)|최대 지정한 횟수만큼 호출|

## 인자 캡쳐
단위 테스트를 실행하다보면 모의 객체를 호출할 때 사용한 인자를 검증해야 할 때가 있다.
Mockito의 ArgumentCaptor를 사용하면 메서드 호출 여부를 검증하는 과정에서 실제 호출할때 전달한 인자를 보관할 수 있다.
``` java
public class UserRegisterMockTest {
    private UserRegister userRegister;
    private EmailNotifier mockEmailNotifier = Mockito.mock(EmailNotifier.class);

    @Test
    void whenRegisterThenSendMail() {
        userRegister.register("id", "pw", "email@email.com");

        ArgumentCaptor<String> captor = ArgumentCaptor.forClass(String.class);
        BDDMockito.then(mockEmailNotifier).should().sendRegisterEmail(captor.capture());

        String realEmail = captor.getValue();
        assertEquals("email@email.com", realEmail);
    }
}
```