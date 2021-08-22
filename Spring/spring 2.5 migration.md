## gradle 6 -> 7 버전
spring boot 2.5 부터 gradle 7.0 버전을 사용하면서 http protocol에 대해서 기본적으로 지원해주지 않는 이슈

allowInsecureProtocol = true 옵션으로 해결

``` gradle
repositories {
  maven {
    url = uri("http://nexus.nhnent.com/content/groups/public/")
    allowInsecureProtocol = true
  }
  mavenCentral()
}
```
compile 옵션을 이제 지원하지 않아 compileOnly로 변경하여 해결

```gradle
dependencies {
  /** Log 수집 */
  compileOnly 'org.json:json:20171018'
  implementation "com.toast.java.logncrash.logback:logncrash-java-sdk3:3.0.4"
}
```
