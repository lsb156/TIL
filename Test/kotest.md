# Kotest

## Test Styles
### BDD Style
#### BehaviorSpec Style
Given, When, Then 전형적인 BDD Style
``` kotlin
class RegisterEmoticonFeature : BehaviorSpec() {
    init {
        Given("본인 인증된 사용자가 로그인된 상황에서") {
            // ...
            When("검수 정보를 입력란에") {
                // ...
                And("검수 정보를 입력하고 검수 등록 버튼을 누르면") {
                    // ...
                    Then("등록 결과가 포함된 검수 진행 목록 화면으로 이동한다.") {
                        // ...
                    }
                }
                And("검수 정보를 입력하지 않고 검수 등록 벝튼을 누르면") {
                    // ...
                    Then("검수 등록 실패 사유가 화면에 표시되어야 한다") {
                        // ...
                    }
                }
            }
        }
    }
}
```
장점으로는 해당 코드의 중복이 최소화 되고 상위에서 사용되는 변수를 그대로 사용이 가능함이 장점

#### FeatureSpec Style
Given When Then 스타일이 아닌 Feature / Scenario 형태로 구조를 지원하는 BDD Style
Given When Then 이전에 시나리오를 작성하게 되는데 더 구조화 하지 않고 테스트 케이스를 작성할때 사용
행위자를 특정하지 않고 그 기능만 테스트를 수행하고자 할때 (ex. xx월 xx일 xx시에 메일 보내기)
``` kotlin
class EmoticonFeature : FeatureSpec() {
    init {
        feature("이모티콘 검수 이메일 발송"){
            // ...
            scenario("""
                2020-08-05 11:00:00(KST)에 전날 생성된 이모티콘 목록을 
                검수자에게 이메일 발송 되어야 한다.
            """) {
                // ...
            }
        }
    }
}
```

### TDD Style
#### AnnotationSpec Style
JUnit 형태의 Testcase 작성을 하게 해주는 TDD용 Style
``` kotlin
 class AccountServiceSpec : AnnotationSpec() {
     @BeforeAll
     fun setupStub() {
        // ...
     }
     
     @AfterAll
     fun clearStub() {
        // ...
     }
     
     @Test
     fun takeAccountIfExistByToken() {
        // ...
     }
}
```

#### ExpectSpec Style
DSL로 Testcase 작성을 하게 해주는 TDD용 Style
``` kotlin
class EmoticonServiceSpec : ExpectSpec() {
    init {
        context("이모티콘 생성을 할 때") {
            // ...
            expect("계정과 이모티콘 정보, 이미지가 있으면 이모티콘이 생성된다.") {
                // ...
            }
        }
    }
}
```
JUnit에서 Testcase 1개당 `expect` 1개
`expect`를 그룹핑하는게 `context`개념

> 출처 : [kotest가 있다면 TDD 묻고 BDD로 가!](https://if.kakao.com/session/106)
