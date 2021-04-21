# HTTP
**H**yper**T**ext **T**ransfer **P**rotocol
- 클라이언트 서버 구조로 전달
- Stateful, Stateless
- connectionless
- HTTP 메시지


## 전달할 수 있는 타입
- HTML, TEXT
- IMAGE, 음성, 영상, 파일
- JSON, XML (API)
- 거의 모든 형태의 데이터 전송 가능

## HTTP 역사
- `HTTP/0.9` 1991년: GET 메서드만 지원, HTTP 헤더X
- `HTTP/1.0` 1996년: 메서드, 헤더 추가
- `HTTP/1.1` 1997년: 가장 많이 사용, 우리에게 가장 중요한 버전
  - RFC2068 (1997) -> RFC2616 (1999) -> RFC7230~7235 (2014)
- `HTTP/2` 2015년: 성능 개선
- `HTTP/3` 진행중: TCP 대신에 UDP 사용, 성능 개선

## Stateless Protocol
- 서버가 클라이언트의 상태를 보존X
- 장점: 서버 확장성 높음(스케일 아웃)
- 단점: 클라이언트가 추가 데이터 전송

### Stateless의 실무 한계
- 모든 것을 무상태로 설계 할 수 있는 경우도 있고 없는 경우도 있다.
- 로그인한 사용자의 경우 로그인 했다는 상태를 서버에 유지
- 일반적으로 브라우저 쿠키와 서버 세션등을 사용해서 상태 유지
- 상태 유지는 최소한만 사용


## Connectionless
- HTTP는 기본이 연결을 유지하지 않는 모델 일반적으로 초 단위의 이하의 빠른 속도로 응답
- 1시간 동안 수천명이 서비스를 사용해도 실제 서버에서 동시에 처리하는 요청은 수십개 이 하로 매우 작음
  - 예) 웹 브라우저에서 계속 연속해서 검색 버튼을 누르지는 않는다.
- 서버 자원을 매우 효율적으로 사용할 수 있음

### Connectionless의 한계
- 매 요청마다 TCP/IP 연결을 새로 맺어야 함 (3 way handshake 시간 추가)
- 웹 브라우저로 사이트를 요청하면 HTML 뿐만 아니라 자바스크립트, css, 추가 이미지 등 수 많은 자원이 함께 다운로드
- 지금은 HTTP 지속 연결(Persistent Connections)로 문제 해결
  - HTTP/2, HTTP/3에서 더 많은 최적화

## HTTP Message
### HTTP Message
#### HTTP Request
``` http request
# Request
GET /search?q=hello&hl=ko HTTP/1.1 
Host: www.google.com

```
#### HTTP Response
``` http request
# Response
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8 Content-Length: 3423

<html> 
    <body>...</body>
</html>
```

#### HTTP 공식 스펙
https://tools.ietf.org/html/rfc7230#section-3
```
start-line
*( header-field CRLF )
CRLF
[ message-body ]
```

```
[start-line] HTTP/1.1 200 OK
[header]Content-Type: text/html;charset=UTF-8 Content-Length: 3423
[empty line (CRLF)]
[message body] <html> <body>...</body></html>
```
