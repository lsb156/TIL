# Static 변수에 Value 사용

```yaml
name: ssabae
```

```java
@Component
public class StaticValueComponent {

    @Value("${name}")
    String name;

    @Value("${name}")
    static String staticName;

    @Value("${name}")
    private void setValue(String name){
        staticName = name;
    }
}
```
