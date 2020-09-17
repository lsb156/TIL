# Spring Transactional

## Isolation
Transaction의 일관성을 나타내는 격리 수준을 명시한다.
자세함 사항은 [Transaction Isolation Level](../Database/Transaction%20Isolation%20Level.md) 참조

Recommand되는 방식으로는 Read시에는 `REPEATABLE READ`, Create, Update, Delete 때는 `SERIALIZABLE`

### DEFAULT
기본 격리수준으로 DB의 기본 Isolcation Level을 따라감

### READ_UNCOMMITTED (Level 0)
- 커밋되지 않는 데이터에 대한 읽기를 허용
- `Dirty Read` 발생

### READ_COMMITTED (level 1)
- 커밋된 데이터만 읽기 허용
- `Dirty Read` 방지한다.
- 하나의 Transaction 안에서 Select시마다 값이 다를수가 있는 `Non-Repeatable Read` 문제 발생

### REPEATABLE_READ (level 2)
- 트랜젝션이 완료될떄까지 select에서 사용하는 모든 데이터에 shared lock이 걸림 (다른곳에서 수정 불가능)
- 트랜젝션 내부에서 일관성있는 데이터 보증
- `Non-Repeatable Read` 방지

### SERIALIZABLE (level 3)
- 데이터의 일관성 및 동시성을 위해 MVCC(Multi Version Concurrency Control) 사용하지 않음
- 트랜잭션이 완료될 때까지 SELECT 문장이 사용하는 모든 데이터에 shared lock이 걸리므로 다른 사용자는 그 영역에 해당되는 데이터에 대한 수정 및 입력이 불가능하다.
- `Phantom Read` 방지



## Propagation
트랜잭션 동작 도중 다른 트랜잭션을 호출(실행)하는 상황에 선택할 수 있는 옵션
`@Transactional`의 propagation 속성을 통해 피호출 트랜잭션의 입장에서는 호출한 쪽의 트랜잭션을 그대로 사용할 수도 있고, 새롭게 트랜잭션을 생성할 수도 있다.


### REQUIRED
Default 속성으로 부모 트랙젝션이 있는 경우 참여하고 없는경우에는 새로운 트랜젝션을 생성한다.
보통 이 설정으 대부분 문제는 해결된다.

### SUPPORTS
이미 시작된 트랜잭션이 있으면 참여하고 그렇지 않으면 트랜잭션 없이 진행
트랜젝션이 없어도 트랜젝션의 경계에서 Connection이나 하이버네이트의 Session을 공유할 수 있다.

### MANDATORY
REQUIRED와 비슷하게 이미 시작된 트랜잭션이 있으면 참여한다.
반면에 트랜잭션이 시작된 것이 없으면 새로 시작하는 대신 예외를 발생시킨다.
혼자서는 독립적으로 트랜잭션을 진행하면 안 되는 경우에 사용

### REQUIRES_NEW
부모의 트랜젝션에 상관없이 항상 새로운 트랜잭션을 시작한다.
이미 진행 중인 트랜잭션이 있으면 트랜잭션을 잠시 보류시킨다.

### NOT_SUPPORTED
트랜잭션을 사용하지 않게 한다.
이미 진행 중인 트랜잭션이 있으면 보류시킨다.

### NEVER
트랜잭션을 사용하지 않도록 강제한다.
이미 진행 중인 트랜잭션도 존재하면 안된다 있다면 예외를 발생시킨다.

### NESTED
이미 진행중인 트랜잭션이 있으면 중첩 트랜잭션을 시작한다.
중첩 트랜잭션은 트랜잭션 안에 다시 트랜잭션을 만드는 것이다.
하지만 독립적인 트랜잭션을 만드는 REQUIRES_NEW와는 다르다.
중첩된 트랜잭션은 먼저 시작된 부모 트랜잭션의 커밋과 롤백에는 영향을 받지만 자신의 커밋과 롤백은 부모 트랜젝션에게 영향을 주지 않는다.

해당 작업은 로그에 대한 정보를 쌓을때 유용하게 사용할 수 있다.
메인 트랜젝션이 끝나기전에 로그를 꼭 DB에 저장하여야 할때 로그 DB가 저장에 실패하더라도 부모의 원래 트랜젝션에 영향을 주어서는 안된다.
하지만 로그를 남긴 후 부모의 트랜젝션이 실패했다면 같이 Rollback 되어야 하는 상황이기에 해당 작업에서는 NESTED가 적합하다.

> 중첩 트랜잭션은 JDBC 3.0 스펙의 저장포인트(savepoint) (클릭시 자세한 내용링크) 를 지원하는 드라이버와 DataSourceTransactionManager 를 이용할 경우에 적용 가능하다.

## Read-Only
트랜잭션을 읽기 전용으로 설정
성능을 최적화하기 위해 사용할 수도 있고 특정 트랜잭션 작업 안에서 쓰기 작업이 일어나는 것을 의도적으로 방지하기 위해 사용할 수도 있다.
일반적으로 읽기 전용 트랜잭션이 시작된 이후 INSERT, UPDATE, DELETE 같은 쓰기 작업이 진행되면 예외가 발생한다.

하이버네이트를 사용하는 경우에는 FlushMode를 Manual로 변경하여 dirty checking을 생략

DB에 따라 DataSource의 Connection 레벨에도 설정되어 약간의 최적화가 가능

> 일부 트랜잭션 매니저의 경우 읽기전용 속성을 무시하고 쓰기 작업을 허용할 수도 있기 때문에 주의

## Rollback
### rollbackFor, noRollbackFor, rollbackForClassName, noRollbackForClassName
특정 예외가 발생 시 강제로 Rollback 시킬지의 여부를 설정함으로
앞에 no가 붙으면 해당 예외는 롤백에 포함시키지 않는다.


## Timeout
어노테이션에 지정한 시간까지 해당 메소드 수행이 완료되지 않는 경우에는 rollback을 실행한다.
지정된 값이 -1일 경우에는 Timeout 시간없이 무한정 대기한다.