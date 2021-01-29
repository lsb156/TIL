# PING 요청을 막는 3가지 방법

- `Security Group`에서 ICPM을 제한하는 방법
- `proc/sys/net/ipv4/icmp_echo_ignore_all`을 활용하여 disable 시키는 방법
- `ipTables` NetFilter에서 만든 방화벽으로 블락

서버단에서 뭔가를 설정한다던가 커널 파라미터를 설정하는것은 지양해야 한다.
버전별로 스크립트들이 틀려지기 때문에 무슨일이 벌어질 지 모른다.
`Security Group` 으로 막는것이 가장 최선책
