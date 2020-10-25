
# Encapsulation (캡슐화)
- 데이터와 관련 기능을 묶는다
- 객체가 어떻게 구현했는지에 대해서 외부에 감추어 내부에 데이터들이 구현에 사용되었는지 감춘다.
- 정보은닉 (Information Hiding) 의미 포함
- 외부의 영향 없이 객체 내부 구현 변경 가능

## 캡슐화의 기능
절차적인 코드를 줄여주고 해당 코드의 의도를 직관적으로 보여줌으로 코드 해석에 대한 시간이 줄어 듦

## 캡슐화를 하는 규칙
### Tell Don't Ask
해당 객체에 정보를 얻지 말고 메시지를 전달하여 요청을 보내는 법칙
``` kotlin
// anti-pattern
if (acc.getMembership() == REGULAR) {
}

// good
if (acc.hasRegularPermission()) {
}
```
### Demeter's Law (데미테르의 법칙)
- 메서드에서 생성한 객체의 메서드만 호출
- 파라미터로 받은 객체의 메서드만 호출
- 필드로 참조하는 객체의 메서드만 호출

``` kotlin
// bad
acc.getExpDate().isAfter(now())

// bad
var date = acc.getExpDate();
date.isAfter(now())

// good
acc.isExpired()

// good
acc.isValid()
```

