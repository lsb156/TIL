# DNS 확인 과정

1. 우선, 로컬의 DNS Cache를 확인
2. `/etc/hosts` 파일에 정적으로 설정한 정보를 확인
3. `/etc/resolv.conf`에 설정한 정보를 기반으로 DNS 서버에게 질의
4. DNS 서버는 정보가 있으면 반환하고 없으면 본인의 상위 DNS에게 질의를 하여 정보 습득
5. 도메인에 해당하는 IP를 알게되면 DNS Cache에 추가


두개의 터미널을 띄워놓고 아래와 같은 순서대로 입력했을 경우에는 패킷이 날아가지 않음
```
# A
$ sudo systemd-resolve --flush-caches
$ nslookup google.com
$ sudo tcpdump -nni eth0 port 53

# B
$ nslookup google.com
```

다시 dns의 캐시를 지우고 날렸을 경우에는 DNS 질의하는 패킷이 검출된다.
```
# A
$ sudo systemd-resolve --flush-caches
$ sudo tcpdump -nni eth0 port 53

# B
$ nslookup google.com
```

