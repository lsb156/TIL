## Argument Capture

``` java
public class PersonTest {

    private Person person;
    private ArgumentCaptor<Integer> argumentCaptor;

    @Before
    public void setup() {
        person = mock(Person.class);
        argumentCaptor = ArgumentCaptor.forClass(Integer.class);
    }

    @Test
    public void personTest() {
        person.setAge(20);
        verify(person, times(1)).setAge(argumentCaptor.capture());
        
        int arg = argumentCaptor.getValue();

        assertEquals(20, arg);

    }
}
```
해당 argument 캡춰가 가능하여 검증이 가능


> https://m.blog.naver.com/PostView.naver?blogId=writer0713&logNo=221123721638&targetKeyword=&targetRecommendationCode=1
