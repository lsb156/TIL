CORS 상황이 발생되었을 경우 브라우저는 다음과 같은 절차를 사용
- 일반적인 요청에 대해서는 아무런 처리도 하지 않음, 일반적인 요청이라고 하면 다음 사항에 부합되는 요청을 의미함
    - GET, HEAD, POST
        - Request Header에는 다음 속성만 허용:
        - Accept, Accept-Language,  Content-Language,  Content-Type
    - Content-Type은 다음만 허용
        - application/x-www-form-urlencoded
        - multipart/form-data
        - text/plain
- 이런 일반적인 요청이 아닌 경우 브라우저는 접근할 리소스를 가지고 있는 서버에 preflighted 요청을 보냄
    - preflighted 요청은 특별한 목적을 가지는 요청으로 method = OPTIONS 으로 전송
    - OPTIONS 요청을 받은 서버는 Response Header에 서버가 허용할 옵션을 설정하여 브라우저에게 전달.
    - 브라우저는 서버가 보낸 Response 정보를 이용하여 허용되지 않은 요청인 경우 405 Method Not Allowed 에러를 발생시키고, 실제 페이지의 요청은 서버로 전송하지 않음
    - 허용된 요청인 경우 전송

> https://www.popit.kr/cors-preflight-%EC%9D%B8%EC%A6%9D-%EC%B2%98%EB%A6%AC-%EA%B4%80%EB%A0%A8-%EC%82%BD%EC%A7%88/
