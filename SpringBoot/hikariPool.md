```
2021-08-26 03:08:07,677 [http-nio-10009-exec-9] WARN  com.zaxxer.hikari.pool.PoolBase - HikariPool-1 - Failed to validate connection com.mysql.cj.jdbc.ConnectionImpl@3f33de41 (No operations allowed after connection closed.). Possibly consider using a shorter maxLifetime value.
```
## 에러 내용
- Hikari Pool의 connection validate 실패
- 이미 close 된 connection에 operation를 하면 안됨
- Hikari Pool의 `maxLifeTime`을 짧게 가야함
- mysql에 설정된

### HikariPool의 MaxLifeTime
- MaxLifetime은 Hikari pool에서 Connection이 살아 있을 수 있는 시간
- 현재 사용중인 Connection은 종료하지 않고 이미 닫혀 있는 경우에만 제거
- maxLifetime은 최소 30초 이상으로 설정하여야 합니다. (안그러면 180000ms로 설정)
- default 180000ms (= 30분)
- 0으로 설정하면 무한으로 Connection이 살아있음
- maxLifeTime에 2.5%의 변화를 주어 모든 Connection이 한순간에 종료되지 않도록 설정되어있음

### Mysql의 wait_timeout
- wait_timeout: 활동하지 않는 커넥션을 끊을때까지 서버가 대기하는 시간
wait_timeout은 session, global variable 두 개로 구성

## 원인
- JDBC4에서는 validationQuery를 수행하여 Connection을 갱신하지 않음
- Hikari PoolEntry에 Scheduled Event를 걸어 maxLifetime 시간 이후에 강제적으로 Connection을 종료
- global wait_timeout 값이 maxLifetime 보다 짧아서 생긴 현상
- 30분마다 종료하려고 하는데 `이미 닫힌 커넥션에서 무엇을 하려고 하기에 경고 문구가 노출`

> HikariCP Failed to Validate Connection Warning 이야기
> https://jaehun2841.github.io/2020/01/08/2020-01-08-hikari-pool-validate-connection/#hikari-pool에서-connection을-관리하는-방법
