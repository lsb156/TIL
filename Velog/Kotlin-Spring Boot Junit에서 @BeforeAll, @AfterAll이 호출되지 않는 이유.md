JUnit5에서 테스트를 할때 해당 테스트 클래스의 시작과 끝에 단 한번씩만 호출해주는 `@BeforeAll`, `@AfterAll` Annotation이 존재합니다.

해당 Annotation을 실행할때의 지켜야할 항목에 대해서는 `static` 메소드여야 하며 리턴타입이 없는 `void` 형태여야 한다고 명시되어있습니다.

그래서 다음과 같이 테스트 클래스를 작성하여 테스트를 해보았습니다.

``` kotlin
class TestApplicationTests {

    companion object {
        @BeforeAll
        fun beforeAll() {
            println("beforeAll")
        }

        @AfterAll
        fun afterAll() {
            println("afterAll")
        }
    }

    @Test
    fun contextLoads() {
        println("contextLoads")
    }

    @BeforeEach
    fun beforeEach() {
        println("beforeEach")
    }

    @AfterEach
    fun afterEach() {
        println("afterEach")
    }

}
```
테스트에 대한 결과는 아래와 같습니다.
```
...
> Task :test
beforeEach
contextLoads
afterEach
BUILD SUCCESSFUL in 1s
...
```
`companion object` 내부에 명시해준 항목이 제대로 실행되지 않은걸 확인할 수 있습니다.

`static`, `void`모든 조건을 충족하셨다고 생각을 하셨었다면 Kotin에서 companion object가 java의 static으로 변황되는 과정에 대해서 알아보아야 합니다.

``` kotlin
    companion object {
        @BeforeAll
        fun beforeAll() {
            println("beforeAll")
        }

        @AfterAll
        fun afterAll() {
            println("afterAll")
        }
    }
```
위의 companion object가 java로 변경된 코드는 다음과 같습니다.
``` java
   public static final class Companion {
      @BeforeAll
      public final void beforeAll() {
         String var1 = "beforeAll";
         boolean var2 = false;
         System.out.println(var1);
      }

      @AfterAll
      public final void afterAll() {
         String var1 = "afterAll";
         boolean var2 = false;
         System.out.println(var1);
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
```

Java와 Kotlin을 혼용해서 사용할때에 Java에서 Kotlin의 companion object 내부에 접근할떄는 `{className}.Companion.{method}()` 형태로 접근한다는것을 어디선가 보셨던 기억이 있으실겁니다.

하나하나의 형태를 일일이 static으로 만들지 않고 companion object를 하나의 class로 정의하여 사용하는걸 알 수 있습니다.

그래서 우리는 static 메소드를 만들기 위해서 `@JvmStatic` 이라는 메소드를 추가적으로 정의해주어야 합니다.

``` kotlin
    companion object {
        @BeforeAll
        @JvmStatic
        fun beforeAll() {
            println("beforeAll")
        }

        @AfterAll
        @JvmStatic
        fun afterAll() {
            println("afterAll")
        }
    }
```
위 kotlin 코드를 자바로 변경해준 소스는 아래와 같습니다.

``` java
   @BeforeAll
   @JvmStatic
   public static final void beforeAll() {
      Companion.beforeAll();
   }

   @AfterAll
   @JvmStatic
   public static final void afterAll() {
      Companion.afterAll();
   }

   public static final class Companion {
      @BeforeAll
      @JvmStatic
      public final void beforeAll() {
         String var1 = "beforeAll";
         boolean var2 = false;
         System.out.println(var1);
      }

      @AfterAll
      @JvmStatic
      public final void afterAll() {
         String var1 = "afterAll";
         boolean var2 = false;
         System.out.println(var1);
      }

      private Companion() {}

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
```

실행하면 아래와 같이 기대한대로 출력이 되는것을 확인할 수 있습니다.
```
...
> Task :test
beforeAll
beforeEach
contextLoads
afterEach
afterAll
BUILD SUCCESSFUL in 2s
...
```