## docker Compose Version 별 yml 변수

### Versioning

`docker-compose.yml` 상단에 작성하는 versioning 작성 방법

* `Version 1`에서는 버저닝을 생략
* `Version 2`부터 마이너 버전(2.x)까지 설정해야 함 (생략시 2.0)
* `Version 3`은 도커 스웜과 같이 사용되도록 디자인 됨

### 각 docker 버전 별 사용 가능한 `docker-compose.yml` 버전

| Compose file format | Docker Engine release | Compose Version |
| :---: | :---: | :---: |
| 3.8 | 19.03.0+ | 1.26.0+ |
| 3.7 | 18.06.0+ | - |
| 3.6 | 18.02.0+ | - |
| 3.5 | 17.12.0+ | - |
| 3.4 | 17.09.0+ | - |
| 3.3 | 17.06.0+ | - |
| 3.2 | 17.04.0+ | - |
| 3.1 | 1.13.1+ | - |
| 3.0 | 1.13.0+ | 1.10.0+ |
| 2.4 | 17.12.0+ | 1.21.0+ |
| 2.3 | 17.06.0+ | 1.16.0+ |
| 2.2 | 1.13.0+ | 1.13.0+ |
| 2.1 | 1.12.0+ | 1.9.0+ |
| 2.0 | 1.10.0+ | 1.6.0+ |
| 1.0 | 1.9.1.+ |  |

### Compose 버전에 따른 차이점

* Structure 허용하는 설정 키값
* 실행해야하는 최소 Docker Engine 버전
* 네트워킹과 관련된 Compose의 동작

### Compose 마이그레이션

#### Version 2.x -> 3.x

Compose 버전 `2`와 `3`의 구조는 동일하지만 삭제된 매개변수들이 있어서 `3.X`에 맞게 수정하여준다.

* `volume_driver`

``` yaml
version: "3.9"
services:
  db:
    image: postgres
    volumes:
      - data:/var/lib/postgresql/data
volumes:
  data:
    driver: mydriver
```

* 서비스에서 볼륨 드라이버를 설정하는 대신 `top-level volumes option`을 사용하여 볼륨을 정의 하고 드라이버를 지정
* `volumes_from`: 서비스간에 볼륨을 공유하려면 `top-level volumes option`을 사용하여 정의하고 서비스 수준 볼륨 옵션을 사용하여 공유하는 각 서비스에서 참조
* `cpu_shares`, `cpu_quota`, `cpuset`, `mem_limit`, `memswap_limit`는 `3.x`의 `resource`로 대체
    * 해당 옵션은 docker swarm에만 적용
        * 컨테이너에 적용하기 위해선 아래 링크와 같이 적용 필요
            * [https://docs.docker.com/compose/compose-file/compose-file-v2/#cpu-and-other-resources](https://docs.docker.com/compose/compose-file/compose-file-v2/#cpu-and-other-resources)
            * [https://github.com/docker/compose/issues/4513](https://github.com/docker/compose/issues/4513)

``` yaml
version: "3.9"
services:
  redis:
    image: redis:alpine
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 50M
        reservations:
          cpus: '0.25'
          memory: 20M
```

* `extends`은 제거되어 다른 방법을 찾아봐야함
    * [https://docs.docker.com/compose/extends/#extending-services](https://docs.docker.com/compose/extends/#extending-services)
* `group_add`은 `3.x`에서 제거됨
* `pids_limit`은 `3.x`에 도입되지 않음 (삭제와 무슨 차이지..?)
* networks - `link_local_ips` 옵션은 `3.x`에 도입되지 않음

#### Version 1 -> 2.x

* 모든것을 한단계씩 들여쓰기를 하고 `services:`를 삽입
* 최상단에 version: `2.x` 지정
* `dockerfile`은 `build` 하위로 옮긴다.

``` yaml
build:
  context: .
  dockerfile: Dockerfile-alternate
```

* `log_driver`, `log_opt`는 `logging` 하위로 두고 다음과 같이 변경한다.

``` yaml
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

* `net`는 `network_mode`로 대체

``` yaml
net: host    ->  network_mode: host
net: bridge  ->  network_mode: bridge
net: none    ->  network_mode: none

net: "container:web"  ->  network_mode: "service:web"

# net: "container:[container name/id]" 형태는 값을 변경할 필요가 없음
net: "container:cont-name"  ->  network_mode: "container:cont-name"
net: "container:abc12345"   ->  network_mode: "container:abc12345"
```

* `volumes` named volumes이 있는 경우 `top-level volumes option`에서 `data`를 선언해야 함

``` yaml
version: "2.4"
services:
  db:
    image: postgres
    volumes:
      - data:/var/lib/postgresql/data
volumes:
  data: {}
```

* 기본적으로 프로젝트 이름으로 볼륨을 선언하지만 `data`라는 이름으로 볼륨을 생성하고 싶으면 다음과 같이 외부로 선언을한다.

``` yaml
volumes:
  data:
    external: true
```

## 버전별 특징 및 구성

### Version.1

[https://docs.docker.com/compose/compose-file/compose-file-v1/](https://docs.docker.com/compose/compose-file/compose-file-v1/)

* yml 문서에 버전명이 생략
* `volumes`, `networks` or `build arguments` 사용 불가
    * networking이 지원되지 않음
    * 모든 컨테이너는 기본 bridge 네트워크 에 배치 되고 해당 IP 주소의 다른 모든 컨테이너에서 연결 가능
    * 컨테이너 간 검색을 활성화 하려면 `links` 를 사용

> links
> [https://docs.docker.com/compose/compose-file/compose-file-v1/#links](https://docs.docker.com/compose/compose-file/compose-file-v1/#links)

``` yaml
web:
  build: .
  ports:
    - "5000:5000"
  volumes:
    - .:/code
  links:
    - redis
redis:
  image: redis
```

### Version.2

[https://docs.docker.com/compose/compose-file/compose-file-v2/](https://docs.docker.com/compose/compose-file/compose-file-v2/)

* `Compose 1.6.0` 이상 `Docker Engine of version 1.10.0+.`에서 동작
* yml 문서에 버전명을 마이너 버전까지 작성 `ex) 2.1`
* 모든 서비스는 services 키 아래에 선언
* `volumes`에 Named volumes 생성 가능
* `networks`에 Network 생성 가능
* 기본적으로 모든 컨테이너는 애플리케이션 전체의 기본 네트워크에 연결되며 서비스 이름과 동일한 호스트 이름에서 검색 가능
  이것은 링크가 거의 불필요하다는 것을 의미.

**links 동작 원리**
이 컨테이너의 /etc/hosts 파일에 그 내용이 추가되어서 컨테이너에서 다른 컨테이너 들에 접근할 수 있게 된다.

``` bash
192.168.1.186 db
192.168.1.186 database
192.168.1.187 redis
```

> 네트워크
> [https://docs.docker.com/compose/networking/](https://docs.docker.com/compose/networking/)

**Version2 예시**

``` yaml
version: "2.4"
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    networks:
      - front-tier
      - back-tier
  redis:
    image: redis
    volumes:
      - redis-data:/var/lib/redis
    networks:
      - back-tier
volumes:
  redis-data:
    driver: local
networks:
  front-tier:
    driver: bridge
  back-tier:
    driver: bridge
```

추가된 매개변수

* network - `aliases`
* Network - `depends_on`:

**depends\_on**

``` yaml
version: "2.4"
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

실행시 `depends_on`에 명시된 `db`, `redis`가 뜬 뒤에 해당 서비스가 실행

### Version 2.1

추가된 매개 변수

* `link_local_ips`
* build, service - `isolation`
* volumes, networks, build - `labels`
* volumes - `name`
* `userns_mode`
* `healthcheck`
* `sysctls`
* `pids_limit`
* `oom_kill_disable`
* `cpu_period`

### Version 2.2

추가된 매개 변수

* `init`
* `scale`
* `cpu_rt_runtime`, `cpu_rt_period`
* build - `network에`

### Version 2.3

추가된 매개 변수

* build - `target`, `extra_hosts`,`shm_size`
* healthcheck - `start_period`
* service - `runtime`
* volumns - `LONG SYNTAX` 제공
* `device_cgroup_rules`

**LONG SYNTAX**

``` yaml
 volumes:
   - type: volume
     source: mydata
     target: /data
     volume:
       nocopy: true
   - type: bind
     source: ./static
     target: /opt/app/static
```

### Version 3

[https://docs.docker.com/compose/compose-file/compose-file-v3/](https://docs.docker.com/compose/compose-file/compose-file-v3/)
삭제된 매개 변수

* `volume_driver`
* `volumes_from`
* `cpu_shares`
* `cpu_quota`
* `cpuset`
* `mem_limit`
* `memswap_limit`
* `extends`
* `group_add`

추가된 매개 변수

* [`deploy`](https://docs.docker.com/compose/compose-file/compose-file-v3/#deploy)

### Version 3.1

추가된 매개 변수

* [`secrets`](https://docs.docker.com/compose/compose-file/compose-file-v3/#secrets)

### Version 3.2

추가된 매개 변수

* [build](https://docs.docker.com/compose/compose-file/compose-file-v3/#build) - `cache_from`
* [`ports`](https://docs.docker.com/compose/compose-file/compose-file-v3/#ports), [`volume`](https://docs.docker.com/compose/compose-file/compose-file-v3/#volumes) 에 `LONG SYNTAX` 지원
* network driver option - [`attachable`](https://docs.docker.com/compose/compose-file/compose-file-v3/#attachable))
* deploy - [`endpoint_mode`](https://docs.docker.com/compose/compose-file/compose-file-v3/#endpoint_mode)
* deploy - [`placement preference`](https://docs.docker.com/compose/compose-file/compose-file-v3/#placement)

### Version 3.3

추가된 매개 변수

* [build - `labels`](https://docs.docker.com/compose/compose-file/compose-file-v3/#build)
* [`credential_spec`](https://docs.docker.com/compose/compose-file/compose-file-v3/#credential_spec)
* [`configs`](https://docs.docker.com/compose/compose-file/compose-file-v3/#configs)

### Version 3.4

추가된 매개 변수

* build - [`target`](https://docs.docker.com/compose/compose-file/compose-file-v3/#target), [`network`](https://docs.docker.com/compose/compose-file/compose-file-v3/#network)
* [healthcheck](https://docs.docker.com/compose/compose-file/compose-file-v3/#healthcheck) - `start_period`
* [update](https://docs.docker.com/compose/compose-file/compose-file-v3/#update_config) \- order
* [volumes](https://docs.docker.com/compose/compose-file/compose-file-v3/#volume-configuration-reference) \- name

### Version 3.5

추가된 매개 변수

* service - [`isolation`](https://docs.docker.com/compose/compose-file/compose-file-v3/#isolation)
* networks, secrets, configs - `name`
* build- [`shm_size`](https://docs.docker.com/compose/compose-file/compose-file-v3/#shm_size)

### Version 3.6

추가된 매개 변수

* [`tmpfs`](https://docs.docker.com/compose/compose-file/compose-file-v3/#tmpfs)
* volume mount type - `tmpfs`

### Version 3.7

추가된 매개 변수

* service - [`init`](https://docs.docker.com/compose/compose-file/compose-file-v3/#init)
* deploy - [`rollback_config`](https://docs.docker.com/compose/compose-file/compose-file-v3/#rollback_config)
* `root of service`, `network`, `volume`, `secret`에 [extension-fields](https://docs.docker.com/compose/compose-file/compose-file-v3/#extension-fields) 제공

### Version 3.8

추가된 매개 변수

* [placement](https://docs.docker.com/compose/compose-file/compose-file-v3/#placement) - [`max_replicas_per_node`](https://docs.docker.com/compose/compose-file/compose-file-v3/#max_replicas_per_node)
* [config](https://docs.docker.com/compose/compose-file/compose-file-v3/#configs-configuration-reference), [secret](https://docs.docker.com/compose/compose-file/compose-file-v3/#secrets-configuration-reference) - `template_driver` option (swarm에 배포시에만 적용)
* secret - [`driver`, `driver_opts`](https://docs.docker.com/compose/compose-file/compose-file-v3/#configs-configuration-reference) option (swarm에 배포시에만 적용)
