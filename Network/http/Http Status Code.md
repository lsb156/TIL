# Http Status Code

클라이언트가 보낸 요청의 처리 상태를 응답에서 알려주는 기능
- 1xx (Informational): 요청이 수신되어 처리중
- 2xx (Successful): 요청 정상 처리
- 3xx (Redirection): 요청을 완료하려면 추가 행동이 필요
- 4xx (Client Error): 클라이언트 오류, 잘못된 문법등으로 서버가 요청을 수행할 수 없음
- 5xx (Server Error): 서버 오류, 서버가 정상 요청을 처리하지 못함

![http-status-ranges](../../asset/Network/http/http-status-ranges.png)

## Status
### 1xx (Informational)
요청이 수신되어 처리중 상태
잘 사용하지 않는 상태

### 2xx (Successful)
클라이언트의 요청을 성공적으로 처리
- 200 OK
  - 요청성공
- 201 Created
  - 요청 성공해서 새로운 리소스가 생성됨
- 202 Accepted
  - 요청이 접수되었으나 처리가 완료되지 않았음 (배치 처리 같은 곳에서 사용)
- 204 No Content
  - 서버가 요청을 성공적으로 수행했지만, 응답 페이로드 본문에 보낼 데이터가 없음
  - save 버튼의 결과로 아무 내용이 없어도 된다.
  -  save 버튼을 눌러도 같은 화면을 유지해야 한다


