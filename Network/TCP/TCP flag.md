# TCP Flag

TCP(Transmission Control Protocol)는
- 3-WAY Handshake 방식을 통해 두 지점 간에 세션을 연결
- 4-WAY Handshake를 통해 세션을 종료

세션연결, 해제 이외에도 데이터를 전송, 거부, 세션 종료 기능이 패킷의 FLAG 값에 표시 됨

FLAG 순서
```
+-----+-----+-----+-----+-----+-----+
| URG | ACK | PSH | RST | SYN | FIN |
+-----+-----+-----+-----+-----+-----+
```
각각 1비트로 TCP 세그먼트 필드 안에 Control BIT 또는 FLAG BIT 로 정의


## TCP Flag 6가지
- URG (Urgent: 긴급데이터)
- ACK (Acknowledgement: 응답)
- PSH (Push: 밀어넣기)
- RST (Reset: 재 연결 종료)
- SYN (Synchronization: 동기화)
- FIN (Finish: 연결 요청 종료)


### SYN (- S)
TCP 에서 세션을 성립할 때  가장먼저 보내는 패킷
시퀀스 번호를 임의적으로 설정하여 세션을 연결하는 데에 사용되며 초기에 시퀀스번호를 보낸다.

### ACK (-Ack)
상대방으로부터 패킷을 받았다는 걸 알려주는 패킷, 다른 플래그와 같이 출력되는 경우도 있습니다.
받는 사람이 보낸 사람 시퀀스 번호에 TCP 계층에서 길이 또는 데이터 양을 더한 것과 같은 ACK를 보냅니다.(일반적으로 +1 하여 보냄) ACK 응답을 통해 보낸 패킷에 대한 성공, 실패를 판단하여 재전송 하거나 다음 패킷을 전송한다.

### RST (- R)
재설정(Reset)을 하는 과정
양방향에서 동시에 일어나는 중단 작업
비 정상적인 세션 연결 끊기에 해당
이 패킷을 보내는 곳이 현재 접속하고 있는 곳과 즉시 연결을 끊고자 할 때 사용

### PSH (- P)
TELNET 과 같은 상호작용이 중요한 프로토콜의 경우 빠른 응답이 중요
이 때 받은 데이터를 즉시 목적지인 OSI 7 Layer 의 `Application Layer`으로 전송하도록 하는 FLAG. 
대화형 트랙픽에 사용되는 것으로 버퍼가 채워지기를 기다리지 않고 데이터를 전달
데이터는 버퍼링 없이 바로 위 계층이 `Application Layer`로 바로 전달한다.

### URG(- U)
`Urgent pointer` 유효한 것인지를 나타낸다.
> Urgent pointer란?
> 전송하는 데이터 중에서 긴급히 전당해야 할 내용이 있을 경우에 사용
> 긴급한 데이터는 다른 데이터에 비해 우선순위가 높아야 한다.
> EX) ping 명령어 실행 도중 Ctrl+c 입력

### FIN(- F)
세션 연결을 종료시킬 때 사용되며 더이상 전송할 데이터가 없음을 나타냄

> https://mindgear.tistory.com/206
