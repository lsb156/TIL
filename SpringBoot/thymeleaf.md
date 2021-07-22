## 조건문

### if 문
``` html
<div th:if="${condition}"> 컨디션에 따라 표시 유무 </div>

<span th:if="${avatar.getName().toString().equals('TEST')}"> 
    아바타 이름이 TEST이면 출력 
</span>
```

### else 문
``` html
<span th:if="${avatar.getName().toString().equals('TEST')}"> TEST </span>
<span th:unless="${avatar.getName().toString().equals('TEST')}"> NOT TEST </span>
```

### switch 문
스위치문은 아래와 같이 작성합니다.
``` html
<h1 th:switch="${param.actionType}">
    <span th:case="buyTicket"> 입장권 구매  </span>
    <span th:case="giftTicket"> 입장권 선물하기  </span>
    <span th:case="*"> 게임 아바타 구매하기 </span>
</h1>
```
위에서부터 내려오다가 조건에 맞는 하나만 보여지게 되어있는 구문입니다.
case에 `*`은 java의 switch 문 중 `default`에 해당하는 표현입니다.

하지만 여기서 `param.actionType`이 `buyTicket`, `giftTicket`과 값이 같은지 비교하고있습니다.
그런데 우리가 원하는 기능은 `param.actionType`만이 아닌 다른값을 이용해서 statement를 작성하고 싶습니다.
예를들어 중간에 `param.traceId`가 `AVLOTTO`와 같을때의 구분이 더 들어갔으면 합니다.

`thymeleaf`는 `if`, `else if`, `else`가 연속된 형태의 구문을 지원해주지 않지만 다음과 같이 switch문을 이용한 꼼수로 비슷하게 표현이 가능합니다.
``` html
<h1 th:switch="true">
    <span th:case="${#strings.equals(param.traceId, 'AVLOTTO')}"> 입장권 구매 </span>
    <span th:case="${#strings.equals(param.actionType, 'buyTicket')}"> 입장권 구매  </span>
    <span th:case="${#strings.equals(param.actionType, 'giftTicket')}"> 입장권 선물하기  </span>
    <span th:case="*"> 게임 아바타 구매하기 </span>
</h1>
```





### sequence
``` html
<th:block th:each="num : ${#numbers.sequence(1,5)}">
  <div th:text="${num}"></div>
</th:block>


<!-- 결과 -->
<div>1</div>
<div>2</div>
<div>3</div>
<div>4</div>
<div>5</div>
```
