# Nginx
무중단 배포가 가능
웹서버, 리버스 프록시, 캐싱, 로드 밸런싱, 미디어 스트리밍 등을 위한 오픈소스

> **리버스 프록시**
> 외부의 요청을 받아 백엔드 서버로 요청을 전달
> 리버스 프록시 서버(Nginx)는 요청을 전달하고 실제 요청에 대한 처리는 백엔드 어플리케이션이 처리

Nginx를 이용한 운영 프로세스
1. 사용자는 서비스 주소로 접속 (80, 443)
2. Nginx는 사용자의 요청을 받아 현재 연결된 :8081(ver 1.0)에 요청을 전달
3. :8082(ver 1.0)는 Nginx 연결되어 있지 않아 요청을 받지 못하는 상태
4. :8082(ver 1.1)로 배포
5. 정상적으로 배포된뒤 :8082(ver 1.1) 정상구동 중인지 확인
6. :8082(ver 1.1)가 정상적으로 판단되면 Nginx는 reload 명령어를 통해 :8082로 연결하고 :8081 연결 해제 (0.1초 걸림)
7. 1,2 버전 배포가 필요하면 4번 부터 반복

``` graphLR
GitHub --> TravisCI
TravisCI --1.jar upload--> AWS_S3
TravisCI --2.request deploy--> AWS_CodeDeploy
AWS_S3 --3.jar upload--> AWS_CodeDeploy
AWS_CodeDeploy --4.deploy--> AWS_EC2
AWS_EC2 --> SpringBoot:8081
AWS_EC2 --> SpringBoot:8082
Browser(사용자) --> Nginx
Nginx --> SpringBoot:8081
Nginx --> SpringBoot:8082
```
