# iptables

## 테이블(tables)

iptables에는 테이블이라는 범주가 있다.

테이블은 아래와 같이 4가지로 나뉜다.
- filter 
- nat 
- mangle 
- raw

## 체인(chain)

iptables에는 filter 테이블에 미리 정의된 세가지의 체인이 존재

- INPUT : 호스트 컴퓨터를 향한 모든 패킷
- OUTPUT : 호스트 컴퓨터에서 발생하는 모든 패킷
- FORWARD : 호스트 컴퓨터가 목적지가 아닌 모든 패킷, 즉 라우터로 사용되는 호스트 컴퓨터를 통과하는 패킷
  
체인은 네트워크 트래픽(IP 패킷)에 대하여 정해진 규칙들을 수행한다.

가령 들어오는 패킷(INPUT)에 대하여 허용(ACCEPT)할 것인지, 거부(REJECT)할 것인지, 버릴(DROP)것인지를 결정한다.
