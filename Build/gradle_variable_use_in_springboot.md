# Gradle 변수를 SpringBoot에서 사용하기

``` groovy
build.gradle
ext {
    name = 'ssabae'
    email = "llsb156@gmail.com"
}

processResources {
    filesMatching('**/application.yml') {
        expand(project.properties)
    }
}
```

```yml
ext:
  name: ${ext.name}
  email: ${ext.email}
```

``` java
@Value("${ext.name}")
private String name;

@Value("${ext.email}")
private String email;
```

## 동작 원리
build 시에는 다음과 같은 task들을 수행하는데
```shell
$ ./build
Executing task 'build'...

:bootBuildInfo
:compileJava
:processResources   <- here!
:classes
:bootJar
:jar SKIPPED
:assemble
:compileTestJava
:processTestResources NO-SOURCE
:testClasses
:test
:check
:build
```
`processResources` 부분에서 `ext`관련 값들을 매핑 시켜줄때 `gradle`에 명시된 값들로 채워줌


그래서 `build` 후에 compile된 `application.yml`파일을 보면 다음과 같이 변경되어있다.
```yaml
# build/resources/main/application.yml
ext.name: ssabae
ext.email: llsb156@gmail.com
```


## 참고 
> https://nevercaution.github.io/spring-boot-use-gradle-value/
> https://docs.spring.io/spring-boot/docs/current/reference/html/howto.html#howto-properties-and-configuration
