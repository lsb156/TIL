# Test Database In Docker

Local 개발 환경에서 Docker를 이용한 테스트 DB 생성 스크립트

## H2, Redis 설치
```
docker pull oscarfonts/h2
docker pull redis
docker pull redis:alpine
```

## 일반적인 LOCAL 데이터베이스 사용
```
cd {project-dir}
mkdir ./redis
mkdir ./h2
cd h2
touch {database-name}.mv.db
touch {database-name}.trace.db
cd ..
docker run -d -p 1521:1521 -p 81:81 -v `pwd`/h2/:/opt/h2-data --name local-test-h2 oscarfonts/h2
docker run -d -p 6379:6379 -v `pwd`/redis:/data --name local-test-redis redis:alpine redis-server
```


## 공용 네트워크 사용시
```
docker network create {network-name}
docker run -d -p 1521:1521 -p 81:81 -v `pwd`/h2/:/opt/h2-data --network {network-name} --name {project-name-h2} oscarfonts/h2
docker run -d -p 6379:6379 -v `pwd`/redis:/data --network {network-name} --{project-name-redis} redis:alpine redis-server
```
