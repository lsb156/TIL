# Spring Boot 2.4에서 변경된 사항들

## 버전명
`2.3.7.RELEASE` 형태로 제공되던 버전의 형태가 `2.4.0`으로 심플하게 변경되었습니다.
Spring Boot `2.4.0` 이상의 버전부터 접미사를 삭제하여야 합니다.

Spring Cloud이 기존에 제공하였던 `Greenwich`, `Hoxton` 방식의 버전명을 제거하고
Calander 형식으로 제공하여 `2020.0.0` 형태의 버전명으로 제공됩니다.



## JUnit Vintage 삭제
JUnit이 5버전만 제공되며 기존에 JUnit4를 사용 할 수 있던 옵션인 Vintage가 제거 되었습니다.
따라서 JUnit4를 계속 사용하실 경우에는 아래와 같이 추가적인 디펜던시를 추가해야 합니다.
``` xml
<!-- maven -->
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
``` groovy
// Gradle
testImplementation("org.junit.vintage:junit-vintage-engine") {
    exclude group: "org.hamcrest", module: "hamcrest-core"
}
```

## application.properties, application.yml
상당히 많이 변경되어 이 챕터가 아닌 다른 챕터에 담을 예정입니다.
이전의 설정을 그대로 사용하고 싶을 경우에는 아래와 같이 설정파일에 명시해주면 이전 버전과 같이 사용이 가능합니다.
``` properties
spring-config.use-legacy-processing=true
```
## spring.cloud.location
외부에서 사용하는 설정을 사용할 경우에는 `spring.cloud.location`를 명시해주어야 합니다. 
`Optional`이 아닌 `Mandantory`로 설정이 되어있어서 생기는 오류인데 `Optional`로 변경이 필요한 경우에는
`spring.cloud.location:optional:{url}` 처럼 `Optional`하게 변경이 가능합니다.

## H2, HSQL, Derby
원래는 스프링부트에서 사용할 DB나 Datasource를 입력하지 않은 경우 자동으로 메모리 디비로 만들어 주었는데
이제는 직접 입력해야 합니다.

datasource를 만들때 해주던 `sa`는 이제 기본값으로 생성해주지 않으며 `spring.datasource.username=sa` 처럼 명시를 해주어야 합니다.

아래와 같이 명시하여 이전 설정처럼 동작하게 할 수 있습니다.
``` properties
spring.datasource.initialization-mode=always
```

## Logback 속성명 변경
다음 Spring Boot 속성이 변경되었습니다.

- `logging.pattern.rolling-file-name` → `logging.logback.rollingpolicy.file-name-pattern`
- `logging.file.clean-history-on-start` → `logging.logback.rollingpolicy.clean-history-on-start`
- `logging.file.max-size` → `logging.logback.rollingpolicy.max-file-size`
- `logging.file.total-size-cap` → `logging.logback.rollingpolicy.total-size-cap`
- `logging.file.max-history` → `logging.logback.rollingpolicy.max-history`

매핑되는 시스템 환경 속성은 다음과 같습니다.

- `ROLLING_FILE_NAME_PATTERN` → `LOGBACK_ROLLINGPOLICY_FILE_NAME_PATTERN`
- `LOG_FILE_CLEAN_HISTORY_ON_START` → `LOGBACK_ROLLINGPOLICY_CLEAN_HISTORY_ON_START`
- `LOG_FILE_MAX_SIZE` → `LOGBACK_ROLLINGPOLICY_MAX_FILE_SIZE`
- `LOG_FILE_TOTAL_SIZE_CAP` → `LOGBACK_ROLLINGPOLICY_TOTAL_SIZE_CAP`
- `LOG_FILE_MAX_HISTORY` → `LOGBACK_ROLLINGPOLICY_MAX_HISTORY`

## Default Servlet
기본으로 제공되는 Setvlet은 `DispatcherServlet`만 사용이되며 `DefaultServlet`을 사용하기 위해서는 추가로 설정을 해주어야 합니다.
```properties
server.servlet.register-default-servlet=true
```

## Flyway 최소 버전
`Flyway 5`를 사용하는 경우 릴리스에 대한 스키마 업그레이드 만 지원되며 Spring Boot 2.4를 사용할 경우에는 `Flyway 6`으로 업그레이드가 필요합니다.

## ConfigurationProperties
`@ConstructorBinding`를 이용하여 기존에 필수였었던 `Setter`를 명시 안해도 됩니다.
