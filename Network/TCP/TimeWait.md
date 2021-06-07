# TIME_WAIT

`TIME_WAIT` 이란 TCP 상태의 가장 마지막 단계
## TIME_WAIT은 왜 길게 설정되어있나?
1. 지연 패킷이 발생할 경우
    - 이미 다른 연결로 진행되었다면 지연 패킷이 뒤늦게 도달해 문제가 발생
    - 다른 연결의 SEQ 까지 동일하다면 잘못된 데이터를 처리하게 되고 데이터 무결성 문제가 발생
2. 원격 종단의 연결이 닫혔는지 확인해야할 경우
    - 마지막 `ACK` 유실시 상대방은 `LAST_ACK` 상태에 빠지게 되고 새로운 `SYN` 패킷 전달시 `RST`를 리턴
    - 이후 새로운 연결들은 오류를 내며 실패하게됨
    - 이미 연결을 시도한 상태로 인지되어 상대방에게 접속 오류 메시지가 출력됨
    
위에 두 오류가 날 경우가 있어 반드시 `TIME_WAIT`은 일정 시간 남아서 패킷의 오동작을 막는게 좋다.
기본으로 구성되어있는 `TIME_WAIT`의 타임아웃 정보는 커널 헤더 `include/net/tcp.h`에 60초로 하드코딩 되어있다.

> https://github.com/torvalds/linux/blob/master/include/net/tcp.h#L120

```
// include/net/tcp.h
#define TCP_TIMEWAIT_LEN (60*HZ) /* how long to wait to destroy TIME-WAIT
                                  * state, about 60 seconds     */
```


## TIME_WAIT 성능문제
다수의 `TIME_WAIT`은 과연 시스템 성능 저하를 가져올까.

> Each socket in TIME_WAIT consumes some memory in the kernel, usually somewhat less than an ESTABLISHED socket yet still significant. A sufficiently large number could exhaust kernel memory, or at least degrade performance because that memory could be used for other purposes.
>
> http://stackoverflow.com/a/1854196

> Because application protocols do not take TIME-WAIT TCB distribution into account, heavily loaded servers can have thousands of connections in TIME-WAIT that consume memory and can slow active connections. In BSD-based TCP implementations, TCBs are kept in mbufs, the memory allocation unit of the networking subsystem[9]. There are a finite number of mbufs available in the system, and mbufs consumed by TCBs cannot be used for other purposes such as moving data. Some systems on high speed networks can run out of mbufs due to TIME-WAIT buildup under high connection load. A SPARCStation 20/71 under SunOS 4.1.3 on a 640 Mb/s Myrinet[10] cannot support more than 60 connections/sec because of this limit.
>
> http://www.isi.edu/touch/pubs/infocomm99/infocomm99-web/

여기서 주의해야 할 점은 `TIME_WAIT`으로 인한 성능 저하 논문은 1997년에 출판
그 당시 메모리는 512MB이고 요즘은 64GB이다.


`struct tcp_timewait_sock` 은 168byte이다.

```
struct tcp_timewait_sock {
    struct inet_timewait_sock tw_sk;
    u32    tw_rcv_nxt;
    u32    tw_snd_nxt;
    u32    tw_rcv_wnd;
    u32    tw_ts_offset;
    u32    tw_ts_recent;
    long   tw_ts_recent_stamp;
};
```
4만개의 `TIME_WAIT`이 있더라도 10MB 이며 아웃바운드 커넥션도 2.5MB만 필요할뿐 요즘은 그다지 성능에 큰 영향을 끼치진 않는다.

## 서버의 소캣 갯수 제한

서버의 소켓 수는 할당 가능한 로컬 포트 만큼인 최대 `65,535`개로 알려져있지만 사실은 그렇지 않다.
서버가 할당하는 것은 포트가 아닌 소켓이며 서버의 포트는 최초 `bind()` 시에 하나만 사용한다.

로컬 포트를 할당하는 것은 클라이언트이며 클라이언트가 `connect()` 시 로컬 포트를 임의로 바인딩하면서 서버의 소켓과 연결된다.

소켓은 `<protocol>`, `<src addr>`, `<src port>`, `<dest addr>`, `<dest port>`이 5개 값이 유니크하게 생성된다.
따라서 서버 포트가 추가되거나 클라이언트의 IP가 추가될 경우 그 만큼의 새로운 쌍을 생성할 수 있어 `TIME_WAIT`가 많이 있어도 문제가 없다.

소켓의 수는 설정된 리눅스 파일 디스크립터만큼 생성할 수 있다.

`$ sysctl fs.file-max` 명령어로 생성 가능한 소켓의 갯수 확인이 가능하다.
``` bash
$ sysctl fs.file-max
fs.file-max = 1610106
```

### 클라이언트의 경우
클라이언트는 포트를 이용해서 접속을 하기 때문에 `port_range`가 최대의 접속 가능한 갯수이다.

```bash
$ sysctl net.ipv4.ip_local_port_range
net.ipv4.ip_local_port_range = 32768  60999
```

그래서 1:1 대용량 접속이 발생할 경우 가능한 최대 요청은 `500 RPS` 정도이다.
> 500 * 60 (TIME_WAIT DEFAULT VALUE) = 30,000

이 수치를 넘어선다면 클라이언트의 로컬 포트가 고갈 될 것이며 `TIME_WAIT` 상태를 재사용 해야한다.

## 재사용
1. 서버에 `TIME_WAIT` 상태가 남아있으며, 클라이언의 로컬 포트가 고갈된 경우
    - 클라이언트 입장에서 서버에 남아있는 `TIME_WAIT` 상태를 알 수 없다.
    - 클라이언트는 계속해서 임의의 포트(ephemeral port)에 `SYN` 패킷을 보낸다. 임의의 포트는 순차 증가하는 형태로 FIFO 기준을 따른다.
    - 서버에는 TIME_WAIT 상태로 남아 있지만 동일 소켓이 `SYN`을 수신하면 재사용하게 된다
    - 양쪽 별도의 설정은 필요 없다.
2. 클라이언트에 TIME_WAIT 상태가 남아있으며, 클라이언트의 로컬 포트가 고갈된 경우 
    - 이 경우는 오류가 발생하여 더이상 접속이 불가능.
    - 소켓은 `<protocol>`, `<src addr>`, `<src port>`, `<dest addr>`, `<dest port>` 5개 값으로 구성되며 로컬 포트가 고갈되면 더 이상 유니크한 값을 만들어 낼 수 없다
    - 클라이언트의 `net.ipv4.tcp_tw_reuse` 옵션을 설정하여 기존 클라이언트에 `TIME_WAIT` 상태로 남아 있던 소켓을 재사용해야 한다
    - ``` shell
      $ telnet 11.22.33.44 5678
      Trying 11.22.33.44...
      telnet: connect to address 11.22.33.44: Cannot assign requested address
      telnet: Unable to connect to remote host: Cannot assign requested address
      ```
3. 클라이언트에 TIME_WAIT 상태가 남아있으며, 클라이언트의 로컬 포트가 고갈되고, 서버의 다른 포트에 접속할 경우
    - 포트가 다를 경우 다시 그만큼의 새로운 소켓 쌍을 만들어낼 수 있음을 뜻한다. 
    - 재사용이 필요 없다
    - ``` shell
      $ ss -tonpa | grep 1033
      TIME-WAIT  0      0              11.22.33.44:5678          11.22.33.44:5000   timer:(timewait,52sec,0)
      ESTAB      0      0              11.22.33.44:5678          11.22.33.44:22     users:(("telnet",29636,3))
      ```
    - 동일한 포트 하나에 하나는 `TIME_WAIT`, 하나는 `ESTABLISHED` 상태
    - 상대방 포트는 다르기 때문에 이렇게 동일한 로컬 포트를 함께 쓰는게 가능


## 타임스탬프
앞서 `TIME_WAIT`이 있어야 할 이유로 일정 시간 남아 있어서 패킷의 오동작을 막아야 한다고 명시되었는데 
그렇게 재사용을 가능하게 하는 옵션이 바로 타임스탬프(`net.ipv4.tcp_timestamps`)이다.

RFC 1323 에서 고성능 익스텐션으로 제안된 옵션 중 하나이며 여기에는 두 개의 4 바이트 타임스탬프 필드가 있다
`net.ipv4.tcp_tw_reuse`를 활성화하면 새로운 타임스탬프가 기존 커넥션의 가장 최근 타임스탬프보다도 큰 경우 `TIME_WAIT` 상태인 커넥션을 재사용한다.
`tcp_tw_reuse`가 비활성화 상태라면 매 번 비어 있는 포트를 스캔하게 되지만, 활성화 상태라면 바로 다음 포트를 사용 또는 재사용한다.


1. 지연 패킷이 발생할 경우
    - 동일한 SEQ라도 이미 지난 타임 스탬프이므로 그냥 버리게된다.
2. 원격 종단의 연결이 닫혔는지 확인해야할 경우
    - 타임스탬프로 해결된다.
    - 서버가 ACK를 받지 못한 상태에서 새로운 커넥션이 SYN을 보내면 타임스탬프를 비교해 무시한다.
    - 위 작업이 이루어 지는 사이 FIN이 재전송된다.
    - 그러면 SYN_SENT 상태에 있던 클라이언트는 RST를 보낸다.
    - 서버가 LAST_ACK 상태를 빠져나온다.
    - 위 작업이 이루어 지는 사이 ACK를 받지 못한 클라이언트는 1초 후 다시 SYN을 전송한다.
    - 서버도 SYN+ACK를 보낸다. 
    - 이제 둘은 정상적으로 `ESTABLISHED` 된다.
    - 타임스탬프가 없으면 오류가 나지만 타임스탬프로 약간의 딜레이만 추가적으로 발생한다.

> 재사용을 위해서는 net.ipv4.tcp_timestamps 타임스탬프 옵션이 서버/클라이언트 양쪽 모두 **반드시 켜져 있어야 한다**

타임스탬프는 옛날에는 CPU 자원을 절약하기 위해 끄기도 했지만 이제는 그런 시대가 아니다.

## 재활용
`TIME_WAIT`을 가장 효율적으로 재활용 하는 방법으로 `net.ipv4.tcp_tw_recycle` 옵션이 있다. 
그러나 NAT 환경에서 문제가 있다

서버가 로드 밸런스 뒤에 위치하는 서비스 환경에선 장비간 타임스탬프가 일치하지 않아
역전 현상이 발생할 경우 패킷 드롭이 발생할 수 있으므로 사용해선 안된다

패킷은 마이크로세컨드 단위로 매우 빠르게 동작하고 장비간 시간을 마이크로 단위로 정확히 맞추기는 사실상 불가능에 가깝기 때문에 사용하기 힘지만 성능은 매우 좋다.

`tcp_tw_recycle` 이 활성화 되어 있으면 `TIME_WAIT` 상태를 `TCP_TIMEWAIT_LEN` 즉, 60초가 아닌 `RTO` 즉, `retransmission timeout` 값으로 적용한다.

> RTO (Retransmission Timeout) :
> 타이머가 작동하는 시간을 의미합니다. 
> 이 값은 기본적으로 RTT에 의해서 계산
>
> RTT (Round Trip  Time) : 
> 네트워크 통신을 하는 두 노드 간에 패킷이 전달되는데 소요된 시간을 의미
> 종단 간 거리가 멀면 멀수록 RTT 값이 커짐 

장비간 타임스탬프를 마이크로 세컨드 단위로 일치시키긴 힘드므로 NAT 환경등에선 사용할 수 없고 반드시 서버/클라이언트가 1:1로 직접 연결된 경우에만 사용해야 한다.


## 정리
`TIME_WAIT`은 패킷의 오동작을 막아주는 우리의 친구같은 존재다.
수 많은 잘못된 정보들 사이에서 아래와 같은 올바른 정보를 반드시 기억해두길 바란다.
- `TIME_WAIT`의 타임아웃은 60초로 하드 코딩되어 있다. 설정할 수 없다.
- 다수의 `TIME_WAIT`이 서버 성능을 저하시킨다는 논문5은 1997년에 출판됐다. 지금은 2016년이다. 20년이 지났다.
- 클라이언트가 서버 투 서버로 한 서버에 요청이 많을 경우 `tcp_tw_reuse` 옵션을 설정해 `TIME_WAIT`을 재사용하도록 한다. 서버는 해당 사항이 없다.
- 오래된 서버인 경우 클라이언트가 서버 투 서버 통신을 많이 한다면 빈 포트 스캔으로 성능 저하가 발생하므로 마찬가지로 tcp_tw_reuse 옵션을 설정한다.
- `tcp_tw_reuse`와 `SO_REUSEADDR`는 서로 다른 소켓에 적용되는 옵션이다.
- 서버/클라이언트 모두 `tcp_timestamps`가 기본값인 켜져 있어야 하며, 끄면 안되고 끌 필요도 없다.
- `net.ipv4.tcp_fin_timeout`은 90 정도로 설정한다.
- `FIN_WAIT1`은 상대방 OS에 문제가 있는 경우다.
- `FIN_WAIT2`는 상대방 어플리케이션에 문제가 있는 경우다.
- `FIN_WAIT2`는 `TIME_WAIT`의 역할을 대행한다.
- 특수한 경우가 아니면 링거 옵션은 사용하지 않는다.
- 서버가 클라이언트를 `accept()` 할때 할당하는 것은 소켓이다. 포트가 아니다.
- 소켓의 최대 갯수는 `65,535`개가 아니다. 소켓은 `<protocol>`, `<src addr>`, `<src port>`, `<dest addr>`, `<dest port>` 5개의 값으로 유니크하게 구성되며, 서버 포트 또는 클라이언트의 IP가 추가될 경우 그 만큼의 새로운 쌍을 생성할 수 있다.

## TL;DR
- 서버는 아무것도 할 필요가 없다.
- 클라이언트는 `net.ipv4.tcp_tw_reuse`를 1로 설정한다.
- 서버와 클라이언트가 NAT 없이 1:1 로 직접 연결되어 있다면 압도적인 성능을 보이는 `net.ipv4.tcp_tw_recycle`을 적극 활용한다.

## diff reuse, recycle 
`tcp_tw_reuse`는 `TIME_WAIT` 상태 커넥션을 다시 사용하기 전에, 최근 연결에 사용한 `timestamp`보다 확실히 커야 포트 재활용 여부를 결정
`tcp_tw_recycle`은 `retransmission timeout(RTO)`으로 `TIME_WAIT` 값을 가지고, 포트 재활용





> http://docs.likejazz.com/time-wait/
