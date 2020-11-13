# Jenkins dood (docker out of docker)

## Jenkins Image Build
기존의 Jenkins Image에 Docker가 설치된 이미지를 만드는 `Dockerfile`을 작성한다.
``` dockerfile
FROM jenkins/jenkins:latest
USER root

# Install the latest Docker CE binaries
RUN apt-get update && \
    apt-get -y install apt-transport-https \
      ca-certificates \
      curl \
      gnupg2 \
      software-properties-common && \
    curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
    add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
      $(lsb_release -cs) \
      stable" && \
   apt-get update && \
   apt-get -y install docker-ce
```

```
$ docker build -t sptek/jenkins-dood:latest .
```
- `docker build` : 지정된 경로에 `Dockerfile`을 찾아서 이미지를 만들어준다. 
- `-t` : 새로운 이미지 생성
- `sptek/jenkins-dood` : 새로 만들어질 이미지명 
- `:latest` : 버전 및 태그명
- `.` : 경로 지정

## Jenkins Run
```
$ docker run -d 
    --name galaxy-to-go-jenkins 
    --restart=unless-stopped 
    -p 9900:8080 -p 9901:50000 
    -v ~/app/jenkins_home:/var/jenkins_home 
    -v /var/run/docker.sock:/var/run/docker.sock 
    sptek/jenkins-dood
```
`-v /var/run/docker.sock:/var/run/docker.sock` 옵션을 꼭 추가하면 도커끼리 socket을 연동시켜주게된다.
소켓끼리 연동이 되면 Jenkins 내부의 도커에서 HostOS에 있는 Docker와 연동이되어 같은 자원을 공유 할 수 있게 된다. 