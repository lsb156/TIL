
## macOS에서 아파치만 docker로 띄워서 사용하기
* local에서 java application을 띄워서 디버깅하고 apache를 띄워서 https프로토콜을 사용하고자 한다.
    * Docker apache -- ajp --> local java application

## Docker Container에서 어떻게 host의 java application을 바라볼 수 있을까?
 * Docker에서 지원하는 network_mode중 host를 사용해서 해결 가능하다.
    * network_mode host : Docker container와 Docker host 간에 네트워크 격리(isolation)를 제거하고 직접 호스트의 네트워킹을 사용합니다.

## mac에서는 network_mode host가 이상하게 동작한다. 해결책은?
* 이상 동작
    * network_mode:host를 사용하면 docker container 자체의 port를 호출하는것이 불가능하다.
        * mac과 리눅스의 환경이 다르기 때문에 결론만 말하면 macOs에서는 network_mode:host를 사용할 수 없다.
* 해결책 실마리
    * https://docs.docker.com/docker-for-mac/networking/#use-cases-and-workarounds
* Docker apache -- ajp --> local java application 호출을 위한 설정
    * Docker-compose
        * network_mode : default
    * apache
        * workers.properies에 설정 추가
            * worker.tomcat.host=host.docker.internal
