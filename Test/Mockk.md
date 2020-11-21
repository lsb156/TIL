# Mockk
Kotlin Mocking Library
mock을 생성하여 어떠한 행위를 할 것인지에 대하여 사전 정의
외부 서비스 테스트시에 사용



## Mockking
테스트를 진행할 클래스가 여러 의존도를 가지고 있는 경우 복잡한 관계로 테스트 코드가 오염되기 쉽다.
그래서 mockking을 하여 사용하여야 한다
``` kotlin
class RegisterEmoticonFeature(_emoticonRepository: EmoticonRepository) : BehaviorSpec() {
    private val accountService = mockk<AccountService>()
    Given("본인 인증된 사용자가 로그인된 상황에서") {
        val account = Account(identified = true)
        every { accountService.take(ofType(String::class)) } 
        return account
    }
}
```
변수를 초기화하고 가짜로 동작해야하는 기능을 every를 이용하여 사전 정의가 가능


## Verify
해당 행위가 원하는대로 시행이 되었는지 검증
``` kotlin
class RegisterEmoticonFeature(_emoticonRepository: EmoticonRepository) : BehaviorSpec() {
    private val accountService = mockk<AccountService>()
    private val emoticonRepository = spyk(_emoticonRepository)
    
    And("검수 정보를 입력하고 검수 등록 버튼을 누르면") {
        val response = emoticonHandler.register(token, request)
        
        Then("등록 결과가 포함된 검수 진행 목록 화면으로 이동") {
            response.authorId shouldBe account.id
            // ...
            verify(exactly = 1) {
                accountService.take(token = token)
                emoticonRepository.save(match<EmoticonEntity> {
                    id.accountId == account.id
                })
            }
        }
    }
}
```
> `shouldBe`는 `kotest`의 `matcher`

`verify(exactly = 1)`은 반드시 1회 호출되어야 함을 의미
value, type 검증이 가능하지만 그렇게 검증하기 위해선 `spyk`로 초기화 해줘야함
verify 내부에서 사용되는 객체들은 모두 mock이여야 하는 제약이 있다.






> 출처 : [kotest가 있다면 TDD 묻고 BDD로 가!](https://if.kakao.com/session/106)
> https://github.com/harry-jk/ifkakao-2020-code