# Socket

socket을 파일을 생성한다는것은 여러가지의 정보를 담고 있다.

- `service` : file descriptor로써 어떤 서비스와 연결되어있는지 나타내는것
- `local ip` :
- `local port`
- `remote ip`
- `remote port` : 없는 포트로 랜덤하게 잡히기 때문에 많은 사용자들이 접속 할 수 있다.


## Max User Processes
socket에 동시에 접속할 수 있는 인원은 해당 어플리케이션이 얼마나 파일을 여러개 만들 수 있는지와 관련있다.
socket에 접속 할대마다 파일을 하나씩 만들어서 처리하기 때문
- `ulimit -a` : soft ulimit
- `ulimit -aH` : hard ulimit


### soft ulimit
```
ubuntu@ip-192-168-10-23:~$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15702
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 15702
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

### hard ulimit
```
ubuntu@ip-192-168-10-23:~$ ulimit -aH
core file size          (blocks, -c) unlimited
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15702
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1048576
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) unlimited
cpu time               (seconds, -t) unlimited
max user processes              (-u) 15702
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

### 자바에서는 왜 max user processes 값을 따라갈까
openjdk 코드에서 `-XX:+MaxFDLimit` 옵션이 true일 경우 setrlimit 으로 limit을 증가
openjdk는 `-XX:+MaxFDLimit`의 default option이 true임

Java에서 동시에 생성 가능한 쓰레드 수는 max user processes를 따라간다.
Java에서 소켓 통신(HTTP API, JDBC 커넥션 등)은 open file 옵션을 따라간다.
단, JDK 내부 코드상에서 hard limit 값이 soft limit에 update된다.
