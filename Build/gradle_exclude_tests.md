# Gradle exclude tests

## 특정 조건 Test에서 제외 시키기
``` groovy
apply plugin: 'java'

test {
    filter {
        // 해당 package test 제외.
        excludeTestsMatching "com.ssabae.exclude.package.*"
        
        // 지정 클래스 test 제외.
        excludeTestsMatching "com.ssabae.ExcludeTest"
        
        // 지정 메소드 test 제외 (excludeTestsMatching)
        excludeTestsMatching "com.ssabae.ExcludeTest.someTestMethod"
        
        // 지정 메소드 test 제외 (excludeTest).
        excludeTest "com.ssabae.ExcludeTest", "someTestMethod"
        
        
        includeTestsMatching "com.ssabae.exclude.package.*"
        includeTest "com.ssabae.ExcludeTest", "someTestMethod"
    
    }
}
```

## 특정 조건 Test
```groovy
test {
    filter {
        includeTestsMatching "com.ssabae.exclude.package.*"
        includeTest "com.ssabae.ExcludeTest", "someTestMethod"
    }
}

```
