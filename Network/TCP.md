# TCP

## 인터넷 프로토콜 스택의 4계층
- 애플리케이션 계층 - HTTP, FTP
- 전송 계층 - TCP, UDP
- 인터넷 계층 - IP
- 네트워크 인터페이스 계층


## TCP 전송 순서
- (Application) 클라이언트에서 메시지를 생성
- (Application Library) Socket 라이브러리를 이용하여 OS로 전달
- (OS) 메시지에 TCP 정보 wrapping
- (OS) IP Packet 정보 wrapping
- (Network Interface) Ethernet

## TCP/IP Packet
### IP Packet
- Source IP
- Destination IP

### TCP Segment
- Source Port
- Destination Port
- 전송제어
- 순서
- 검증 정보

## 3WAY HandShake
```
SYN: 접속 요청 
ACK: 요청 수락

[Client] --[SYN]--> [Server]

[Client] <--[SYN + ACK]-- [Server]

[Client] --[ACK]--> [Server]

# ACK와 함꼐 데이터 전송 가능
```

## 순서보장
Client에서 Packet을 1,2,3 순서대로 보냈는데 Server에 1 다음에 3이 도착한경우 서버는 2부터 다시 보내라는 명령을 보낸다.


