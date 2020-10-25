## Design Principles and Design Patterns
 모든 개발이 그렇듯 프로젝트 초반에는 완벽하게 설계가 된것 같고 견고하게 구조를 다듬어 나아가는듯하게 개발이 되지만 개발을 하면 할수록 나의 코드와 구조의 틀이 점점 어긋나기 시작합니다.
 어쩔수 없이 리팩토링을 나중에 하지만 그럴때마다 항상 떠오르는 기본중의 기본 원칙, 바로 이번에 소개하는 `객체지향 개발 5대 원칙`을 항상 떠올리게됩니다.
 시간이 없어 마저 다하지 못한 리팩토링을 덮어놓고 이번 프로젝트를 마무리하면서 뼈저리게 생각하게한 이 원칙들을 머릿속에 다시한번 정리하고 싶어 블로그를 남겨봅니다.

 **해당 포스트에 사용되는 코드는 Kotlin으로 작성되었습니다.**

### 1. SRP (단일책임의 원칙: Single Responsibility Principle)
> There should never be more than one reason for a class to change

 클래스는 한가지 기능만 가지며 한가지 책임을 수행하는데 집중되어야 하는게 SRP가 의미하는 원칙입니다.
 클래스가 변경되어야 할 여러 가지 이유가있는 경우는 클래스에 너무 많은 책임을 부여하고 있음을 의미하기도 합니다.
 SRP 원칙을 따르면 클래스를 여러가지로 분할하여 유연하게 설계할 수 있는 장점을 가지고있으며 다른 행동, 책임이 격리 되어있어 연쇄적인 사이드이펙트가 발생할 여지가 줄어들며 그만큼 수정될 코드 또한 적어지는 장점이 있지만 너무 많은 분할로 인하여 책임이 여러군데로 파편화 되어있는 경우에는 산탄총수술로 다시 응집력을 높여주는 작업이 추가로 필요할 수 있습니다.

 > **Low Coupling, High Cohesion**
 - 여러가지의 책임으로 나눌때는 각 책임간에 결합도를 최소로 하여야 한다. (**Low Coupling**)
 - Shotgun surgery(산탄총 수술) : 하나에 여러가지 책임이 있는것의 반대인 상황으로 하나의 책임이 여러군데 분산되어있는 상황, 하나의 수정 사항이 여러군데에 영향을 미치는 경우 다시 전부 하나로 모아주어 설계를 깔끔하게 유지 시켜주도록 한다. (**High Cohesion**)

 Low Coupling, High Cohesion 항목에 대해서는 [GRASP (object-oriented design)](https://velog.io/@lsb156/GRASP-object-oriented-design) 설계 원칙에 있는 항목으로 낮은 결합도, 높은 응집력을 의미합니다.

 위에 설명드렸던 의미들을 코드로 표현해 보고자 다음과 같이 구현해보았습니다.

``` kotlin
class BigMac {
    var brand = "Macdonald"
    var price = 4900
    var calorie = 525f
    var patty: BigMacPatty = BigMacPatty()
    var bun: SesameBread = SesameBread()
}
```
간단하게 모두 다 아실법한 빅맥을 구현해보았지만 이 클래스에는 두 가지 책임이 존재하게됩니다.
햄버거의 구성을 나타내는 `재료`들과 통계 및 정산시에 사용되는 `정보` 항목이 그렇습니다.

해당 클래스를 SRP에 맞추어 책임을 분산 하여보겠습니다.
``` kotlin
class BigMac {
    var patty: BigMacPatty = BigMacPatty()
    var bun: SesameBread = SesameBread()
    var information: BigMacInformation = BigMacInformation()
}

class BigMacInformation {
    var brand = "Macdonald"
    var price = 3000
    var calorie = 525f
}
```
`정보`에 해당하는 BigMacInformation을 [Extract Class](https://refactoring.guru/extract-class)를 통하여 따로 만들어 줌으로써 분리를 함으로 두가지로 묶여있던 책임을 따로 분리 할 수 있게 되었습니다.
 여러가지의 클래스가 공통적인 기능들로 묶여 있는 경우에는 [Extract Superclass](https://refactoring.guru/extract-superclass)를 이용하여 책임을 추출 하여 한곳에서 관리되도록 설계하는 방법이 효과적입니다.

### 2. OCP (개방폐쇄의 원칙: Open Close Principle)
> Software Entites (Class, Modules, Functions, Etc.) should be open for extension, But closed for modification
소프트웨어의 구성요소(컴포넌트, 클래스, 모듈, 함수)는 확장에는 열려있고, 변경에는 닫혀있어야 한다

  이것은 변경에 대한 비용을 최대한 줄이면서 확장에 대해서는 가능한 극대화 해야 된다는것으로 다른 추가 사항이 일어나더라도 기존 구성은 변경하지 않으며 확장에 대한 가능성을 열어줘야 한다는 뜻입니다

 이를 적용하기 위해서는 각 모듈 간 호출, 의존에 대해서 Concrete Class가 아닌 Interface 또는 추상화에 의존하도록 설계되어야 하며 코드를 매번 수정하지 않고도 새로운 상황에 적응할 수 있도록 대응해야합니다.

그 예시를 밑에서 시나리오를 통하여 확인을 해보겠습니다.
``` kotlin
class BigMac {
    var patty: BigMacPatty = BigMacPatty()
    var bun: SesameBread = SesameBread()
    var information: BigMacInformation = BigMacInformation()
}

class BigMacInformation {
    var brand = "Macdonald"
    var price = 3000
    var calorie = 525f
}
```
위와같이 빅맥을 알차게 구성하였지만 클라이언트의 요구로 다른 햄버거가 추가되어야 한다고 요구사항이 왔습니다.
 지금 상황에서 구조 변경의 개선이 없이 버거를 따로 만든다고 한다면 우리는 높은 확률로 아래와 같이 만들 것입니다.

``` kotlin
class ShanghaiSpicyChickenBurger {
    var patty: ChikenPatty = ChikenPatty()
    var bun: SanghaiBun = SanghaiBun()
    var information: SanghaiInformation = SanghaiInformation()
}
```
 이전에 SRP원칙을 적용하였던 코드를 그대로 사용하기엔 무리가 있어보입니다.

 다른 햄버거들을 구현하면서 나오는 구현체와 클래스의 가짓수와 관리해야 하는 포인트들이 많이 늘어났기 때문입니다.

  기존의 빅맥 클래스가 가지고 있는 구조는 의존도와 결합도가 너무 강하여 이것을 좀 유연하게 바꿀 필요가 있을 것 같다는 생각이듭니다.

 더이상 관리하기 힘들어지기 전에 우리는 여러가지 클래스들의 공통점과 같은 관심사들을 찾아서 한군데에 묶어줘야합니다.

 우리는 이런 변화할 수 있는 가능성을 미리 예측하고 설계하여 미리 적용 하여야합니다.
 개발자는 항상 변화와 확장에 대해서 대비를 하고 설계를 해야한다는 선배들의 말이 문득 떠오릅니다.

 앞서 설명하였던 Concrete Class가 아닌 Interface나 Abstract Class에 의존하도록 설계를 해야한다는 설명에 맞게 빅맥 클래스의 구조를 변경해보겠습니다.
``` kotlin
interface Bun {}

interface Patty {}

interface Information {
    var brand: String
    var price: Int
    var calorie: Float
}

abstract class Hamburger {
    abstract var bun: Bun
    abstract var patty: Patty
    abstract var information: Information
}
```
앞으로 추가될 햄버거에 대해서 공통적으로 사용이 가능한 인터페이스들을 먼저 정의 하여 앞으로 변경이 예상되는 항목들을 미리 정의 함으로써 최소한의 항목만 변경이 가능하도록 설계하여줍니다.
그리고 인터페이스들을 구현할 추상클래스인 Hamburger를 만들어 관리가 가능한 틀 안에 가두도록 합니다.

``` kotlin
class BigMacPatty: Patty {...}
class SesameBread: Bun {...}
class BigMacInformation: Information {
    override var brand = "Macdonald"
    override var price = 3000
    override var calorie = 525f
}
```
각 인터페이스를 구현하는 클래스를 만들어준뒤에 추상화 클래스를 상속받아 만든 여러가지 햄버거들을 다음과 같이 구현하여줍니다.

``` kotlin
class BigMac: Hamburger() {
    override var patty: Patty = BigMacPatty()
    override var bun: Bun = SesameBread()
    override var information: Information = BigMacInformation()
}

class ShanghaiSpicyChickenBurger: Hamburger() {
    override var patty: Patty = ChickenPatty()
    override var bun: Bun = SanghaiBun()
    override var information: Information = SanghaiInformation()
}
```
 추상화에 의존함은 곧 핵심적인 부분만 남기고 불필요한 부분은 자기 클래스 자신의 Context에 적합한 내용들로 채워넣음으로써 각 Context에 적합하게 구체화가 가능합니다.

햄버거라는 공통 추상화를 상속받아 만들고 각 의존하는 클래스들을 인터페이스로 묶음으로써 생기는 이점들은 다음과 같습니다.
- 상하이치킨버거와 그 하위 의존 클래스를 추가하는 것만으로 기능을 확장시킬 수 있다.
- 클래스 추가를 하였지만 기존 소스코드의 수정이 없다.

OCP를 정리하자면 개방-폐쇄가 의미하는 말인즉슨 컴파일 타임의 의존성을 인터페이스에 고정시키고 런타임 의존성을 변경시키게 함으로써 추가클래스를 생성하여도 **동작은 추가**(`개방`) 되지만 기존의 코드는 **수정되지 않음**(`폐쇄`)을 의미합니다.



> 개방 폐쇄 원칙 OCP 은 관리가능(maintainable)하고, 재사용(reusable) 가능한 코드를 만드는 기반이다. 잘 설계된 코드는 기존 코드의 변경 없이 확장 가능하다. OCP 를 가능케 하는 중요 메커니즘은 추상화와 다형성이다. 추상화와 다형성을 가능케 하는 키 메커니즘이 상속이다. 추상 기반 클래스의 순수 가상 함수로 부터 클래스를 상속 파생 시킴으로써 추상화된 다형 인터페이스를 만들어 낼 수 있다.  
    - 로버트 C. 마틴

### 3. LSP (리스코브 치환의 원칙: The Liskov Substitution Principle)
> Subtypes must be substitutable for their base types.
서브 타입은 언제나 기반 타입으로 교체할 수 있어야 한다

 LSP는 상속의 기본적인 메커니즘을 표현하고있습니다.
 LSP는 사용자의 관점에서 기능에 영향을 미치지 않고 서브 클래스를 부모 클래스로 대체 할 수 있어야 합니다.

 간단한 코드로 예시를 들어보겠습니다.

``` kotlin
data class LoginRequest(
    val id: String,
    val password: String
)

class LoginHelper {
    fun login(loginRequest: LoginRequest) {
    	println("NormalLogin : ${loginRequest.id}")
    }
}

class LoginController(val loginHelper: LoginHelper) {
    fun login(loginRequest: LoginRequest) {
        loginHelper.login(loginRequest)
    }
}

fun main() {
    val loginHelper: LoginHelper = LoginHelper()
    val loginController = LoginController(naverLoginHelper)
    val request = LoginRequest("lsb156", "p@ssw0rd")
    loginController.login(request)
}
```
 우리 회사에서는 일반적인 로그인 서비스를 제공하고있던 와중에 네이버나 카카오톡 아이디 로그인을 진행하는 코드가 추가로 필요하다고 가정을 해보겠습니다.

 해당 LSP을 적용한다면 login을 하는 클래스를 상속받아 login을 호출하는 곳을 구현체가 아닌 상위 클래스의 스펙을 가르키고 있도록 변경해야합니다.

 상위 클래스는 하위클래스에서 기본적으로 구현해야할 기본적인 기능만 가지고 있거나 추상화 하여야하며 하위 클래스는 상위클래스에서 정의해놓은 정말 필요한 기능만을 구현하도록 합니다.

 위에서 해놨던 소스에 추가적인 로그인 기능을 가지고 있는 다른 클래스들을 추가하여 보겠습니다.

``` kotlin
class LoginHelper {
    fun login(loginRequest: LoginRequest) {
    	println("NormalLogin : ${loginRequest.id}")
    }
}

class KakaoLoginHelper : LoginHelper() {
    override fun login(loginRequest: LoginRequest) {
        println("KakaoLogin : ${loginRequest.id}")
    }
}

class NaverLoginHelper : LoginHelper() {
    override fun login(loginRequest: LoginRequest) {
        println("NaverLogin : ${loginRequest.id}")
    }
}

class LoginController(val loginHelper: LoginHelper) {
    fun login(loginRequest: LoginRequest) {
        loginHelper.login(loginRequest)
    }
}

fun main() {
    val kakaoLoginHelper: LoginHelper = KakaoLoginHelper()
    // val naverLoginHelper: LoginHelper = NaverLoginHelper()
    val loginController = LoginController(kakaoLoginHelper)
    val request = LoginRequest("lsb156", "p@ssw0rd")
    loginController.login(request)
}

```
 이런 형태로 login하는 행위에 대해서는 변함이 없지만 로그인하는 프로세서가 변경되었습니다.
 실제 프로젝트에서 추가적인 부분이나 변경되어야 하는 상황에 대해서 많은 부분을 수정해야 하는 상황이 생기지만 LSP가 추구하는 원칙대로만 설계한다면 구현체만 변경함으로써 나머지 부분은 그대로 가져갈 수 있는 장점이 있습니다.

 LSP의 다른 예로 Spring에서 JDBC Driver를 보통 Properties에 FQCN(Fully Qualified Class Name) 형태로 작성하고 DB연결을 시도합니다. 하지만 여기서 다른 드라이버로 변경하여도 역시 디비 접속이 잘 됩니다.
 JDBC 기본 Spec에만 초점을 맞추어 서비스를 만들어간다면 어떤 Class가 오는지는 중요하지 않게됩니다.
> 객체지향을 사용하면 아키텍트는 플러그인 아키텍처를 구성할 수 있고,
이를 통해 고수준의 정책을 포함하는 모듈은 저수준의 세부사항을 포함하는 모듈에 대해
독립성을 보장할 수 있다.
저수준의 세부사항은 중요도가 낮은 플러그인 모듈로 만들 수 있고,
고수준의 정책을 포함하는 모듈과는 독립적으로 개발하고 배포할 수 있다.
    - 로버트 C. 마틴

#### 리스코프 치환 원칙의 강제
- 하위형에서 메서드 인수의 반공변성
- 하위형에서 반환형의 공변성
- 하위형에서 메소드는 상위형 메소드에서 지정한 예외 외의 다른 예외를 던지는 것을 비허용
- 하위형에서 선행조건은 강화될 수 없음
- 하위형에서 후행조건은 약화될 수 없음
- 하위형에서 상위형의 불변조건은 반드시 유지되어야 함

#### 공변성 반공변성 무공변성
||의미|
|:-|:-|
|공변성(covariant)|T’가 T의 서브타입이면, `C<T’>`는 `C<? extends T>`의 서브타입이다.|
|반공변성(contravariant)|T’가 T의 서브타입이면, `C<T>`는 `C<? super T’>`의 서브타입이다.|
|무공변성(invariant)|C<T>와 C<T’>는 아무 관계가 없다.|

상속이 있으면 다형성도 같이 가져갈 수 있습니다. 하지만 다형성의 이점을 얻기 위해서는 하위 클래스와 상위 클래스의 클라이언트 간의 규약을 항상 지켜야합니다.
 LSP의 원칙에 따라서 구현을 하게된다면 OCP는 자연스럽게 따라오게됩니다.
 LSP는 원칙을 준수하는 상속구조를 제공함으로써 이를 바탕으로한 OCP 원칙을 통해 확장하는 부분에 다형성을 제공하여 변화에 열려있는 프로그램을 만들 수 있도록 합니다.

### 4. ISP (인터페이스 분리의 원칙: Interface Segregation Principle)
> Client should not be forced to depend upon interfaces that they do not use.
 한 클래스는 자신이 사용하지 않는 인터페이스는 구현하지 말아야 한다.

 어떤 클래스가 다른 클래스를 상속받았을때 최소한의 인터페이스만을 사용해야합니다.
 ISP는 일반적인 한개의 인터페이스보다 구체적인 여러가지의 인터페이스를 구현하는 원칙입니다.
 SRP가 클래스의 단일책임을 강조한다면 ISP는 인터페이스의 단일책임을 강조합니다.
 SRP의 목표는 클래스 분리를 통하여 이루어지고 ISP는 인터페이스 분리를 통하여 이루어집니다.


``` kotlin
interface Arithmetic {
    fun plus(a: Int, b: Int): Int
    fun minus(a: Int, b: Int): Int
    fun times(a: Int, b: Int): Int
    fun div(a: Int, b: Int): Int
}

class Test01 : Arithmetic {
    override fun plus(a: Int, b: Int) = a + b
    override fun minus(a: Int, b: Int) = a - b
    override fun times(a: Int, b: Int) = throw NotSupportedException("times")
    override fun div(a: Int, b: Int) = throw NotSupportedException("div")
}

class Test02 : Arithmetic {
    override fun plus(a: Int, b: Int) = throw NotSupportedException("plus")
    override fun minus(a: Int, b: Int) = throw NotSupportedException("minus")
    override fun times(a: Int, b: Int) = a * b
    override fun div(a: Int, b: Int) = a / b
}

```
해당 상황에서처럼 4칙연산을 담당하게하는 Interface를 만들어놨다고 하고
Test01에서는 plus, minus만 사용하고 Test02에서는 times, div만 사용 할 경우에는
이를 작게 정말 필요한 인터페이스만 구현 가능하도록 변경해야합니다.
``` kotlin
interface Plus { fun plus(a: Int, b: Int): Int }
interface Minus { fun minus(a: Int, b: Int): Int }
interface Times { fun times(a: Int, b: Int): Int }
interface Div { fun div(a: Int, b: Int): Int }

class Test01 : Plus, Minus {
    override fun plus(a: Int, b: Int) = a + b
    override fun minus(a: Int, b: Int) = a - b
}

class Test02 : Times, Div {
    override fun times(a: Int, b: Int) = a + b
    override fun div(a: Int, b: Int) = a - b
}
```
ISP에서 주의해야할 사항은 기존 클라이언트에 변화를 주지 않으면서 인터페이스만을 분리하여 구현해야 한다는 점입니다. 그렇게 인터페이스를 분리함으로서 의존성을 약화시켜 리팩토링 및 구조 변경에 용이하게 만들어줍니다.


### 5. DIP (의존성역전의 원칙: Dependency Inversion Principle)
> - High-level modules should not depend on low-level modules. Both should depend on abstractions.
> - Abstractions should not depend on details. Details should depend on abstractions.
1. 고수준 모듈은 저수준 모듈에 의존해서는 안됩니다. 둘 다 추상화에 의존해야합니다.
2. 추상화는 세부 사항에 의존해서는 안됩니다. 세부 사항은 추상화에 따라 달라집니다.

전통적인 구조에서는 상위 레벨에서 하위 레벨에 의존하게 되어 하위 레벨에서의 변경이 상위 레벨까지 전파하게 되어집니다.
 DIP는 각각의 Class 또는 모듈 간의 의존성을 끊고 상위 레벨에서 정의한 추상을 하위레벨 모듈이 구현하게 하는 원칙으로 외부에서 의존성을 주입받아 Low Coupling을 만들게 하는게 목표입니다.
 이 원칙들을 준수한다면 더 유연해지고 유지 관리가 편해지지만 새로운 클래스 구현 및 추상화 측면에서 복잡성이 발생합니다.

DIP에 대해서는 IoC(Inversion of Control)와 함께 다음에 포스팅으로 추가적으로 설명하도록 하겠습니다.

## 마무리

5대 원칙들이 다들 비슷하지만 요구하는 규칙들이 조금씩 다릅니다.
가끔 이 원칙들이 헷갈리기도 하고 적용하기 힘들었는데 이번에 정리하면서 많은것을 다시 되새길 수 있어 좋았습니다.
 실제 프로젝트 개발에 100% 적용하기엔 어려울지라도 몸에 베여있으면 언젠간 좋은 결실로 다가올것 같습니다.


> 참고내용
http://www.nextree.co.kr/p6960/
http://www.cvc.uab.es/shared/teach/a21291/temes/object_oriented_design/materials_adicionals/principles_and_patterns.pdf
https://refactoring.guru/extract-class
https://refactoring.guru/extract-superclass
