# Amazon EC2 환경 셋팅

``` bash
# SdkMan Dependency 설치
sudo apt install unzip
sudo apt install zip
# SdkMan 설치
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
# SdkMan를 이용한 java 설치
sdk install java 11.0.7-amzn
sdk install java 8.0.252-amzn

# 서버 시간대를 한국으로 변경 
sudo rm /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
```

## 호스트 네임 변경
Ubuntu
``` bash
sudo vim /etc/cloud/cloud.cfg
# preserve_hostname: false # true 변경
hostnamectl set-hostname {호스트이름}
# 이후 프롬프트창에서 원하는 이름 입력
systemctl restart systemd-logind.service
```
Linux
``` bash
sudo vim /etc/sysconfig/network
# HOSTNAME 항목 수정
sudo reboot
```

Hostname이 등록 되었다면 `/etc/hosts` 에 변경한 hostname을 등록하여야 한다. [hostname 미등록으로 인한 장애](http://woowabros.github.io/experience/2017/01/20/billing-event.html)
``` bash
sudo vim /etc/hosts
#127.0.0.1    {등록한 HOSTNAME}
```
이후 `curl {등록한 HOSTNAME}` 명령어를 이용해 정상적으로 등록되었는지 확인이 가능하다.

- 에러 메시지 : **Could not resolve host : {등록한 HOSTNAME}**
- 정상메시지 : **Failed to connect to {등록한 HOSTNAME}**
