
컨테이너 가상화 기술은 서비스간 자원을 격리하는데 별도의 OS를 안 띄워도 되는 장점이 있음
OS 기동시간이 없기 떄문에 자동화시에 매우 빠르고 자원 효율도 높은 서비스가 가능

## Script
### Build
```
# docker -t {Repository}/{Image Name}:{Version} ./Dockerfile
```
`{Version}` 을 명시해주지 않으면 latest로 알아서 만들어줌
`Dockerfile`을 생략하면 알아서 `Dockerfile`을 찾아줌

### Run
```
docker run -d -p 8100:8000 {Repository}/{Image Name}
```
`-d` : 컨테이너 구동을 백그라운드로 시킨다
`-p` : `{Host IP}:{Container IP}` Host Ip와 Contatiner 내부의 아이피를 연동시켜준다.
