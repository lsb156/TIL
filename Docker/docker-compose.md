# Docker Compose

- [Docker Compose란?](#Docker-Compose란?)
- [주요 명령어](#주요-명령어)
- [Docker Compose 설정 방법](#Docker-Compose-설정-방법)
- [Docker Compose Environment](Docker Compose Environment)
- [Compose Network 설정 옵션](#Compose-Network-설정-옵션)
- [Compose Network 설정 옵션](#Compose-Network-설정-옵션)
- [docker Compose Version별 yml 변수](#docker-Compose-Version별-yml-변수)
- [버전별 특징 및 구성](#버전별-특징-및-구성)

# Docker Compose

* Docker Compose란?
* 주요 명령어
* Docker Compose 설정 방법
* Docker Compose Environment
* Compose Network 설정 옵션
* docker Compose Version별 yml 변수
* 버전별 특징 및 구성

## Docker Compose란?

Docker Compose는 다중 컨테이너 애플리케이션을 정의하고 공유할 수 있도록 개발된 도구
YAML 파일을 통하여 단일 명령을 사용하여 모두 실행하거나 모두 종료가 가능

YAML 파일에 애플리케이션 스택을 정의하고 프로젝트 리포지토리 루트에 파일을 저장함으로
다른 사용자가 Repository를 복제하고 `Compose` 실행만으로 구동이 가능함

### 단일 호스트의 여러 격리 된 환경

Compose는 프로젝트 이름을 사용하여 환경을 서로 격리
여러 다른 컨텍스트에서 이 프로젝트 이름을 사용하여 접근한다.
`-p` 옵션을 통하여 프로젝트 이름 변경이 가능하며 Default 값은 프로젝트 폴더 이름이 된다.
`docker-dompose up -p {프로젝트 이름}`

### 컨테이너 생성시 볼륨 데이터 보존

컨테이너 생성시 볼륨 데이터 보존하여 데이터가 휘발되지 않도록 처리

### 변경된 컨테이너 만 재생성

컨테이너를 만드는 데 사용되는 구성을 캐시하여 변경되지 않은 서비스를 다시 시작하면 Compose는 기존 컨테이너를 다시 사용

### 변수 및 환경 간 구성 이동

Compose 파일의 변수를 지원
변수를 사용하여 다양한 환경 또는 다른 사용자에 맞게 컴포지션 커스텀 가능

## 주요 명령어

### up -d

`docker-compose.yml` 파일의 내용에 따라 이미지를 빌드하고 서비스를 실행

#### 진행사항

1. 서비스를 띄울 네트워크 설정
2. 필요한 볼륨 생성(혹은 이미 존재하는 볼륨과 연결)
3. 필요한 이미지 풀(pull)
4. 필요한 이미지 빌드(build)
5. 서비스 의존성에 따라 서비스 실행

#### options

* `-d`: 서비스 백그라운드로 실행. (docker run에서의 -d와 같음)
* `--force-recreate`: 컨테이너를 지우고 새로 생성.
* `--build`: 서비스 시작 전 이미지를 새로 생성

### ps

현재 환경에서 실행 중인 각 서비스의 상태를 표시

``` shell
   Name                  Command               State           Ports
---------------------------------------------------------------------------
nhn_app_1     docker-entrypoint.sh sh -c ...   Exit 1
nhn_mysql_1   docker-entrypoint.sh mysqld      Up       3306/tcp, 33060/tcp
```

### stop, start

서비스를 멈추거나, 멈춰 있는 서비스를 시작

``` shell
$ docker-compose stop
Stopping nhn_mysql_1 ... done

$ docker-compose start
Starting app   ... done
Starting mysql ... done
```

### down

서비스를 삭제한다.
컨테이너와 네트워크를 삭제하며, 옵션에 따라 볼륨도 삭제

``` shell
$ docker-compose down
Stopping nhn_mysql_1 ... done
Removing nhn_app_1   ... done
Removing nhn_mysql_1 ... done
Removing network nhn_default
```

#### options

* `--volume`: 볼륨까지 같이 삭제 (DB 데이터 같이 삭제하는데 용이함)

### exec

실행 중인 컨테이너에서 명령어를 실행

``` shell
$ docker-compose exec django ./manage.py makemigrations

$ docker-compose exec db psql postgres postgres
```

### run

docker run 과 마찬 가지로 특정 명령어를 일회성으로 실행
`exec`는 프로세서를 실행시켜놓을때 사용되고 `run`은 batch성 작업에 사용 특화 된것으로 보임

#### options

* `--rm` : 해당 명령어가 종료된 뒤 컨테이너도 종료

### logs

output으로 나온 log들을 확인 할때 사용
docker의 logs와 마찬가지로 `--follow`, `-f`를 하여 실시간으로 나오는 로그 확인이 가능하다.

``` shell
$ docker-compose logs -f
Attaching to docker_app_1, docker_mysql_1
app_1    | yarn install v1.22.5
app_1    | info No lockfile found.
app_1    | [1/4] Resolving packages...
app_1    | [2/4] Fetching packages...
app_1    | [3/4] Linking dependencies...
app_1    | [4/4] Building fresh packages...
app_1    | success Saved lockfile.
app_1    | Done in 0.14s.
app_1    | yarn run v1.22.5
app_1    | error Couldn't find a package.json file in "/app"
app_1    | info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
mysql_1  | 2021-01-05 04:43:03+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 5.7.32-1debian10 started.
```

#### options

* `-f`, `--follow`: 실시간 로그 출력

### config

`docker-compose`에 최종적으로 적용된 설정을 보여줌
`-f`를 이용하여 여러개의 설정파일을 띄웠을때 확인에 용이함

``` shell
$ docker-compose config
services:
  app:
    command: sh -c "yarn install && yarn run dev"
    environment:
      MYSQL_DB: todos
      MYSQL_HOST: mysql
      MYSQL_PASSWORD: secret
      MYSQL_USER: root
    image: node:12-alpine
    ports:
    - published: 3000
      target: 3000
    volumes:
    - /Users/nhn/project/nhn/hangame-msa/docker:/app:rw
    working_dir: /app
  mysql:
    environment:
      MYSQL_DATABASE: todos
      MYSQL_ROOT_PASSWORD: secret
    image: mysql:5.7
    volumes:
    - todo-mysql-data:/var/lib/mysql:rw
version: '3.8'
volumes:
  todo-mysql-data: {}
```

## Docker Compose 설정 방법

``` yaml
version: "3.8" # 
services:
  web:
    # 웹 애플리케이션 설정
  db:
    # 데이터베이스 설정
networks:
  # 네트워크 설정
volumes:
  # 볼륨 설정
```

> docker-compose version 확인 링크
> [https://docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/)

## 서비스 정의

### App Service 정의

``` shell
docker run -dp 3000:3000 \
  -w /app -v ${PWD}:/app \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

상단의 `Docker` 실행문을 아래처럼 `Docker Compose`로 작성이 가능하다.

``` yaml
version: "3.8"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
```

장점은 상대경로 입력이 가능하다.

### MySQL 서비스 정의

``` shell
docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
```

``` yaml
version: "3.8"

services:
  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

### 최종본

``` yaml
version: "3.8"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment: 
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

`docker-compose.yml` 파일명으로 YAML 파일을 작성하고 `docker-compose up` 명령어를 통하여 시작
백그라운드 실행 옵션 `-d`

``` shell
docker-compose up -d
```

> 만약 `docker-compose.yml`이 아닌 다른 파일을 실행하기 위해선
> `docker-compose -f {YAML_FILE_PATH} up` 형식으로 명령이 가능하다.
>
> `ex) $ docker-compose -f docker-compose-custom.yml up`
> `docker-compose -f docker-compose.yml -f docker-compose-test.yml up` 형태로 두개도 가능하다.
> YAML을 두개 이상 설정할 경우 앞에 있는 설정보다 뒤에 있는 파일이 우선시 된다.

## Docker Compose Environment

### `.env` file

환경변수 설정으로 아래와 같이 참조가 가능하다.

``` yaml
web:
  image: "webapp:${TAG}"
```

기본적으로 `docker-compose up`을 한 상태에서는 `.env` 파일을 찾아서 내부에 있는 값을 환경 변수로 사용한다.

``` yaml
$ cat .env
TAG=v1.5

$ cat docker-compose.yml
version: '3'
services:
  web:
    image: "webapp:${TAG}"

$ docker-compose config

version: '3'
services:
  web:
    image: 'webapp:v1.5'


$ export TAG=v2.0
$ docker-compose config

version: '3'
services:
  web:
    image: 'webapp:v2.0'
```

여러 파일에서 동일한 환경 변수를 설정할 때 사용할 값을 선택하기 위해 Compose에서 사용하는 우선 순위

1. Compose file
2. Shell environment variables
3. Environment file
4. Dockerfile
5. Variable is not defined

### 상황에 따른 `.env` 적용법

`dev`, `prod`, `local` 환경에 따라서
`.env.dev`, `.env.prod`, `.env.local` 형식으로 할당을 줄 수 있는데
`--env-file` 옵션으로 environment 파일 지정이 가능하다.

``` yaml
$ docker-compose --env-file ./config/.env.dev up
```

### CLI Environment

아래의 명령어들은 각각 그 아래의 compose 파일이 일치하게 맵핑된다.

``` yaml
$ docker run -e VARIABLE1

web:
  environment:
    - VARIABLE1
```

``` yaml
$ docker-compose run -e DEBUG=1 web python console.py

web:
  environment:
    - DEBUG=1
```

### Docker Cli Environment Variable

CLI에서 실행할때의 참고 변수 키 값
[https://docs.docker.com/compose/reference/envvars/](https://docs.docker.com/compose/reference/envvars/)

## Compose Network 설정 옵션

``` yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres
    ports:
      - "8001:5432"
```

위 Compose를 실행했을때의 네트워크 과정

1. `myapp_default` 네트워크 생성
2. `web` 컨테이너가 생성. (`web`이라는 이름으로 `myapp_default`에 접속)
3. `db` 컨테이너가 생성. (`db`이라는 이름으로 `myapp_default`에 접속)

* 네트워크가 구성된 이후 `postgres://db:5432` 주소로 접속 가능
* 내부 컨테이너 끼리는 `postgres://db:5432`로 접속하고 외부에선 `postgres://{DOCKER_IP}:8001`로 접속
* 컨테이너 업데이트를 위해 다시 시작할 경우에는 이름은 같지만 다른 IP로 네트워크가 생성되어
  IP 베이스는 다시 찾아서 연결하여야 하고 다른 컨테이너에선 같은 이름으로 재 연결을 시도하여야 한다.

### Links

다른 서비스의 컨테이너에 연결
이름만 지정하거나 `{name}:{alias}`형식으로 지정할 수 있다.
`depend_on` 처럼 디펜던시 관계가 맺어지며 네트워크랑 같이 쓰일때는 하나 이상의 네트워크가 관리되어 네트워크만 사용하길 권장한다.

``` yaml
version: "3"
services:
  web:
    build: .
    links:
      - "redis"
      - "db:database"
  db:
    image: postgres
```

### Custom Network

Custom Network를 통해 좀 더 복잡한 네트워크 토폴로지를 만들 수 있다

``` yaml
version: "3"
services:
  proxy:
    build: ./proxy
    networks:
      - frontend
  app:
    build: ./app
    networks:
      - frontend
      - backend
  db:
    image: postgres
    networks:
      - backend
networks:
  frontend:
    # Use a custom driver
    driver: custom-driver-1
  backend:
    # Use a custom driver which takes special options
    driver: custom-driver-2
    driver_opts:
      foo: "1"
      bar: "2"
```

`top-level network` 옵션에서 커스텀하게 지정이 가능
`app`에서만 `proxy`, `db`에 접근 가능하고 `db`와 `proxy`는 서로 접근이 불가능하다

### IPv4, IPv6

IPv6 사용하려면 `enable_ipv6`를 반드시 `true`로 해야 함

``` yaml
version: "2.4"

services:
  app:
    image: busybox
    command: ifconfig
    networks:
      app_net:
        ipv4_address: 172.16.238.10
        ipv6_address: 2001:3984:3989::10

networks:
  app_net:
    driver: bridge
    enable_ipv6: true
    ipam:
      driver: default
      config:
        - subnet: 172.16.238.0/24
          gateway: 172.16.238.1
        - subnet: 2001:3984:3989::/64
          gateway: 2001:3984:3989::1
```

### Default Network

default를 이용하여 자체 네트워크를 지정하면서 앱 전체의 기본적인 네트워크 구성이 가능

``` yaml
version: "3"
services:

  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres

networks:
  default:
    # Use a custom driver
    driver: custom-driver-1
```

### Use Pre Existing Network

기존의 외부의 네트워크를 사용하려면 다음과 같이 `external`을 사용하여 명시

``` yaml
networks:
  default:
    external:
      name: my-pre-existing-network
```


> 참고
> [https://docs.microsoft.com/ko-kr/visualstudio/docker/tutorials/use-docker-compose](https://docs.microsoft.com/ko-kr/visualstudio/docker/tutorials/use-docker-compose)
> [https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose#%EB%8F%84%EC%BB%A4-%EC%BB%B4%ED%8F%AC%EC%A6%88%EC%9D%98-%EC%A3%BC%EC%9A%94-%EB%AA%85%EB%A0%B9%EC%96%B4](https://www.44bits.io/ko/post/almost-perfect-development-environment-with-docker-and-docker-compose#%EB%8F%84%EC%BB%A4-%EC%BB%B4%ED%8F%AC%EC%A6%88%EC%9D%98-%EC%A3%BC%EC%9A%94-%EB%AA%85%EB%A0%B9%EC%96%B4)
> [https://docs.docker.com/compose/environment-variables/](https://docs.docker.com/compose/environment-variables/)
> [https://docs.docker.com/compose/env-file/](https://docs.docker.com/compose/env-file/)
> [https://docs.docker.com/compose/reference/envvars/](https://docs.docker.com/compose/reference/envvars/)