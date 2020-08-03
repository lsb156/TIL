#Kotlin

<!--[TOC]: # "## Table of Contents"-->

## Table of Contents
- [if](#if)
- [변수](#변수)
- [문자열](#문자열)
- [클래스](#클래스)
- [수열](#수열)
- [이터레이션](#이터레이션)
- [in](#in)
  - [확장 프로퍼티](#확장-프로퍼티)
  - [infix 중위 호출(infix call)](#infix-중위-호출infix-call)
  - [destructuring declaration 구조 분해 선언](#destructuring-declaration-구조-분해-선언)
  - [확장함수](#확장함수)


|한글|영어|설명|
|-|-|-|
|타입 추론|type inference|타입을 정의하지 않아도 컴파일러가 문맥을 구려해 변수 타입을 결정하는 기능|
|순수 함수|pure function|함수형 프로그래밍에서 입력이 같으면 항상 같은 출력을 내놓고 다른 객체의 상태를 변경하지 않으며, 함수 외부나 다른 바깥 환경과 상호작용하지 않는 함수|
|대화형 쉘|REPL|read-eval-print-loop 입력을 받아 값을 계산한 다음 결과 값을 출력하는 루프|
|스마트 캐스트|smart cast|타입 검사 + 타입캐스트 + 타입 강제 변환|
|형 변환|coerce|타입 변환|
|가시성|visibility modifier|public, private, protected|
|뒷받침 필드|backing field|프로퍼트의 값을 저장히가 위한 필드|
|수열|progression|어떤 범위에 속한 값을 일정한 순서로 이터레이션 하는 경우 (1..10)|
|수신 객체 타입|receiver type|확장 함수의 확장이 정의될 클래스의 타입|
|수신 객체|receiver object|확장함수가 호출되는 대상이 되는 값(this)|


## #02 Kotlin Base
### if
kotlin에서 if는 값을 만들어내지 못하는 `statement`가 아닌 결과를 만드는 `expression`이다

### 변수
val (value) - immutable
var (variable) - mutable

### 문자열
문자열 템플릿을 **println("안녕하세요 $name")** 사용 할 경우 `StringBuilder`로 컴파일 된다.

### 클래스
kotlin의 기본 가시성은 public이다

### 수열
``` kotlin
for (i in 1..100) { ... }
for (i in 100 downTo 1 step 2) { ... }
for (i in 1 until 100) { ... }
for (i in 0..size-1) { ... } // 바로 위의 until과 같다.
```
### 이터레이션
``` kotlin
for (c in 'A'..'F') { ... }

val binaryReps = TreeMap<Char, String>()
for ((letter, binary) in binaryReps) { ... }

val list = arrayListOf("10", "11", "100")
for ((index, element) in list.withIndex()) { ... }
```

### in
``` kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'

// kotlni
c in 'a'..'z'
// java
'a' <= c && c <= 'z'
```
## #03 함수 정의와 호출
> 디폴트 값이 정의된 코틀린 함수를 자바에서 호출하기 위해서는 모든 인자를 명시해야한다.
> 자바에서 좀더 편하게 코틀린 함수를 추가하고 싶을 경우 `@JvmOverroads`를 함수에 추가하면 컴파일러가 자동으로 맨 마지막 파라미터로 부터 파라미터를 하나씩 생략한 오버로딩한 자바 메소드를 추가해준다.


> 코틀린 최상위 함수가 포함되는 클래스의 이름을 바꾸고 싶다면 파일에 `@JvmName` 어노테이션을 추가한다. 이 어노테이션은 파일의 맨 앞 패키지 이름 선언 이전에 위치해야 한다.
> ``` kotlin
> // kotlin
> @file:JvmName("StringFunctions")
> package strings
> fun joinToString(...)
>
>// java
>import strings.StringFunctions;
>StringFunctions.joinToString(...)
> ```

#### 확장 프로퍼티
``` kotlin
var StringBuilder.lasChar: Char 
	get() = get(length - 1)
	set(value: Char) {
		this.setCharAt(length - 1, value)
	}
```
#### infix 중위 호출(infix call)
인자가 하나 뿐인 일반 메소드나 인자가 하나뿐인 확장 함수에 중위 호출을 사용할 수 있다.
``` kotlin
infix fun Any.to(other: Any) = Pair(this, other)

val to1 = "test".to(1)
val to2 = "test" to(1)
val to3 = "test" to 1
```
#### destructuring declaration 구조 분해 선언
```
val (str, number) = "test" to 1
```
#### 확장함수
``` kotlin
class User(val id: Int, val name: String, val address: String)
fun User.validateBeforeSave() {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("Can't save user $id: empty $fieldName")
        }
    }
    validate(name, "Name")
    validate(address, "Address")
}

fun saveUser(user: User) {
    user.validateBeforeSave()
    ...
}
```
## #04 클래스 객체 인터페이스
companion object는 private생성자를 호출하기에 아주 좋은 위치에 있다.
``` kotlin
// 일반적인 secondary constructor 여러개가 있는 상황
class User {
	val nickname: String

	constructor(email: String) {
		nickname = email.substringBefore('@')
	}

	constructor(facebookAccountId: Int) {
		nickname = getFacebookName(facebookAccountId)
	}
}

// 의미 있는 이름을 이용하여 생성자를 호출함으로 사용하기 훨씬 편해졌다.
class User private constructor(val nickname: String) {
	companion object {
		fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
		fun newFacebookName(account: id) = User(getFacebookName(accountId))
	}
}
```

