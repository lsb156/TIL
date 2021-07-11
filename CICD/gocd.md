
# GOCD

## 지원하는 기능

### Auth
![img.png](2961629438810921732.png)
file, LDAP 기반의 로그인 환경 구성이 가능하다.


### History
이벤트 발생한 내역들이 기록되어 볼 수 있음.
![img.png](2961627193320678399.png)
회원관련 기능이 아직 적용되어있지 않은 상황이라 `Triggered by annonymous`로 나온다.
클릭하면 저장소 위치, 마지막 커밋된 내역을 볼 수 있다

![img.png](2962311888145026846.png)

회원 관리 적용 이후 화면

## Pipeline 구성
![img.png](2961650725365830567.png)


### Pipeline Group
![img.png](2967055663346935530.png)
파이프라인 그룹 내부에는 여러개의 파이프라인들로 구성이된다.
실제적으로 하나의 프로젝트 배포 단위로 사용이 가능하다.

### Pipeline
![img.png](2967055905078925436.png)
- 하나의 파이프라인 안에 여러개의 Stage들이 존재하고 Stage1이 끝나면 Stage2로 체이닝된다.


### Stage
![img.png](2967056188272964078.png)
Stage는 여러개의 Job을 가지고 있다
Job들은 Stage내에서 병렬로 실행되지만 하나의 Job이 실패하면 그 Stage는 실패처리된다

### Job
![img.png](2967056831021301936.png)

Job은 여러개의 Task를 가지고 있다.
Job 하위의 Task들은 순차적으로 실행된다
이전 task가 실패할 경우 다음 task가 실행되지 않는다
Job 안에 있는 task들은 하나의 agent에서 실행되는 것이 보장된다
Job은 단순한 명령어 뿐만 아니라 플러그인도 사용 가능하다

![img.png](2967097206740025686.png)
주의 : Job은 병렬로 실행이된다.

### Task
![img.png](2967057409524938221.png)
외부에서는 확인할 수 없지만 Job 설정에 들어가서 확인이 가능

최하위 실행단위로 보통 shell script를 실행
Task에서는 Target Agent를 선택해서 지정이 가능하다.
서버에서는 직접적으로 Task를 수행하지 않기때문에 `Build`만 담당하는 Agent를 구분해서 사용해야 할 필요가 있다.

### build-test
![img.png](2961612663985339825.png)

## Install gocd
gocd가 동작하기 위해서는 하나의 Server에 여러 Agent가 붙어서 실행되는 방식이다

### gocd-server
docker에서 실행이 가능하게끔 centos용 image를 따로 제공한다
- https://hub.docker.com/r/gocd/gocd-agent-centos-7
  centos7 환경에서 centos-7전용 image가 아니더라도 linux 공통으로 제공하는 이미지로 실행이 가능하다.
``` yaml
# gocd-server
version: "3.7"
services:
  gocd_server:
    container_name: gocd-server
    restart: always
    ports:
      - 8153:8153
      - 8154:8154
    image: 'gocd/gocd-server:v21.1.0'
    volumes:
      - gocd_server:/godata
volumes:
  gocd_server:
```


#### File Base Auth
gocd에서 사용될 id와 password를 생성
``` bash 
$ sudo htpasswd -B -c ./password.properties goadmin
New password: ***********
Re-type new password: ***********
Adding password for user goadmin

$ cat password.properties
goadmin:$2y$05$L3OB5.8zlu4gj1TLHA7OL.j8HqrHKnQs2dz.wcfyuYHFINXvCxu3i
```

``` bash 
$ docker exec -it gocd-server /bin/bash
$ mkdir /godata/auth
$ vi password.properties
# goadmin:$2y$05$L3OB5.8zlu4gj1TLHA7OL.j8HqrHKnQs2dz.wcfyuYHFINXvCxu3i 입력
```

위와같이 수동으로 작성 또는 도커의 파일복사 기능을 이용해서 copy한다
![img.png](2961749958198777564.png)

위와같이 설정해주면 id/password 기반의 로그인 환경이 구성된다
> LDAP 기반의 설정과 같이 설정이 가능

![img.png](2961780620540832402.png)

### gocd-agent
https://hub.docker.com/r/gocd/gocd-agent-centos-7
> gocd-server의 버전과 gocd-agent의 image버전을 일치시켜주어야 한다
``` yaml 
version: "3.7"
services:
  gocd_agent:
    container_name: gocd-agent
    restart: always
    image: 'gocd/gocd-agent-centos-7:v21.1.0'
    environment:
      AGENT_AUTO_REGISTER_KEY: 8c65d1a5-159b-47a7-b176-b21d7be30c1b
      AGENT_AUTO_REGISTER_HOSTNAME: agent-wa803
      GO_SERVER_URL: http://10.162.5.33:8153/go
      JAVA_HOME: /gocd-jre
    volumes:
      - gocd_agent_godata:/godata
      - gocd_agent_home:/home/go
volumes:
  gocd_agent_godata:
  gocd_agent_home:
```

다음 항목을 채우면 서버에 자동으로 agent 등록이 된다
https://docs.gocd.org/21.1.0/advanced_usage/agent_auto_register.html

|key|required|description|
|:-|:-:|:-|
|AGENT_AUTO_REGISTER_KEY|Y|서버의 agentAutoRegisterKey 값|
|AGENT_AUTO_REGISTER_RESOURCES|N|agent 태그 목록 (쉼표 구분)|
|AGENT_AUTO_REGISTER_ENVIRONMENTS|N|자동 연결에 필요한 environments list (쉼표 구분)|
|AGENT_AUTO_REGISTER_HOSTNAME|N|server에 등록되는 agent 이름|
![img.png](2961784938970315938.png)
`AGENT_AUTO_REGISTER_KEY` 값으로는
`Admin` -> `Config XML` -> 선택영역에 나와있는 `agentAutoRegisterKey` 값을 이용한다.

![img.png](2961888126992734877.png)
정상적으로 agent가 올라간 모습.

#### Agent 내부에 Java 설치
Gocd Agent 자체가 jar를 구동 실행시키는 구조이기 때문에 Java가 이미 적용되어있다.
그래서 docker-compose 실행시에 환경변수에 `JAVA_HOME`만 적용시켜주면 gradle 빌드할때 Java 사용이 가능하다.
``` yaml 
version: "3.7"
services:
  gocd_agent:
    environment:
      JAVA_HOME: /gocd-jre
```

JAVA 적용 상태 확인 방법.
``` bash 
[irteam@hgaa-cloud-wa803 gocd-agent]$ docker exec -it gocd-agent /bin/bash
[go@65094646d892 /]$ $JAVA_HOME/bin/java --version
openjdk 15.0.1 2020-10-20
OpenJDK Runtime Environment AdoptOpenJDK (build 15.0.1+9)
OpenJDK 64-Bit Server VM AdoptOpenJDK (build 15.0.1+9, mixed mode, sharing)
```


