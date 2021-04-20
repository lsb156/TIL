# UDP
User Datagram Protocol

## 특징
- 연결지향적이라 TCP와 같은 3way handshake 같은 기능이 없다.
- 데이터가 잘 전달 되었는지 보증할 수 없다.
- 데이터 전달 순서에 대해서 보증하지 않는다.
- 순서나 데이터 전달에 대해 보증하지 않아 처리 속도가 매우 빠르다.

`IP`와 하는일이 거이 같지만 `IP` + `PORT` + `checkSum` 정도만 추가한다.
