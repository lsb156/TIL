# Offline Pessimistic Lock

A라는 사용자가 수정 화면을 요청할때 해당 Row를 오프라인 잠금상태로 만들고 화면에 대해 응답을 내려준다.
이때 잠금상태라 다른 사용자는 그 화면에 진입을 할 수가 없다.
그리고 수정에 대한 Form이 완성되고 사용자가 수정요청을하여 오프라인 잠금이 해지되고 수정된 처리 결과를 응답해준다.
사용자가 잠금을 요청하고 잠금을 풀지 않는것을 방지하기 위해 유효시간을 설정해 준다.
유효시간이 지난 후 수정요청을 하게 된다면 A라는 사용자가 잠금을 걸었음에도 요청이 실패하게 된다.

``` java
public interface LockManager {
    LockId tryLock(String type, String id) throws LockException;
    void checkLock(LockId lockId) throws LockException;
    void releaseLock(LockId lockId) throws LockException;
    void extendsLockExpiration(LockId lockId, long inc) throws LockException;
}

public class LockId {
    private String value;
    public LockId(String value) { this.value = value; }
    public String getValue() { return this.value; }
}
```
tryLock 메소드에 잠글 대상 Type과 Index를 값으로 전달하여 Lock을 선점한다.
잠금을 해제하거나 잠금이 유효한지 검사하거나 잠금의 유효시간을 늘릴 때 LockId를 사용한다.
LockId는 Controller를 통해 화면단에 데이터로 전달하여 Lock 해제용으로 사용한다.

``` sql
create table locks(
    'type' varchar(255),
    id varchar(255),
    lockid varchar(255),
    expiration_time datetime,
    primary key ('type', id)
) character set utf8

create unique index locks_idx ON locks (lockid);

insert into locks values ('Order', '1', '생성한 lockid', '2020-12-11 09:10:00');
```
type, id 두개의 키 값으로 동시에 두 사용자가 특정 타입의 데이터에 대한 잠금을 구하는것을 방지할 수 있다.

``` java
@Getter 
@AllargumentConstructor
public class LockData {
    private String type;
    private String id;
    private String lockId;
    private String expirationTime;
    
    public boolean isExpired() {
        return expirationTime < System.currentTimeMillis();
    }
}
```
``` java
@Component
public class SpringLockManager implements LockManager {
    private int lockTimeout = 5 * 60 * 1000;
    private JdbcTemplate jdbcTemplate;

    // locks 테이블에서 조회한 데이터를 LockData로 매핑하기 위한 RowMapper
    private RowMapper<LockData> lockDataRowMapper = (rs, rowNum) ->
            new LockData(rs.getString(1), rs.getString(2),
                    rs.getString(3), rs.getTimestamp(4).getTime());

    @Transactional
    @Override
    // Type과 Id에 잠금이 존재하는지 검사
    // 새로운 LockId 생성 (UUID로 유니크한 아이디 생성)
    // 잠금 생성 후 리턴
    public LockId tryLock(String type, String id) throws LockException {
        checkAlreadyLocked(type, id);
        LockId lockId = new LockId(UUID.randomUUID().toString());
        locking(type, id, lockId);
        return lockId;
    }

    // 잠금이 존재하는지 검사
    // type과 id에 대한 데이터를 조회 후 유효 시간 검사
    // 유효 기간이 존재하지 않는 LockData가 존재하면 익셉션 발생  
    private void checkAlreadyLocked(String type, String id) {
        List<LockData> locks = jdbcTemplate.query(
                "select * from locks where type = ? and id = ?",
                lockDataRowMapper, type, id);
        Optional<LockData> lockData = handleExpiration(locks);
        if (lockData.isPresent()) throw new AlreadyLockedException();
    }

    // 유효 시간이 지나면 해당 데이터를 삭제하고 값이 없는 Optional을 리
    private Optional<LockData> handleExpiration(List<LockData> locks) {
        if (locks.isEmpty()) return Optional.empty();
        LockData lockData = locks.get(0);
        if (lockData.isExpired()) {
            jdbcTemplate.update(
                    "delete from locks where type = ? and id = ?",
                    lockData.getType(), lockData.getId());
            return Optional.empty();
        } else {
            return Optional.of(lockData);
        }
    }

    // 턴잠금을 위해 lock 테이블에 데이터 삽입
    // 삽입 결과가 없으면 exception 발생
    private void locking(String type, String id, LockId lockId) {
        try {
            int updatedCount = jdbcTemplate.update(
                    "insert into locks values (?, ?, ?, ?)",
                    type, id, lockId.getValue(), new Timestamp(getExpirationTime()));
            if (updatedCount == 0) throw new LockingFailException();
        } catch (DuplicateKeyException e) {
            throw new LockingFailException(e);
        }
    }

    // 현재 시간 이후로 유효 시간 생성 
    private long getExpirationTime() {
        return System.currentTimeMillis() + lockTimeout;
    }

    // 잠금이 유효한지 검사
    @Override
    public void checkLock(LockId lockId) throws LockException {
        Optional<LockData> lockData = getLockData(lockId);
        if (!lockData.isPresent()) throw new NoLockException();
    }

    // LockId에 해당하는 LockData를 구한다 handleExpiration을 이용해서 유효 시간이 지난 LockData를 처리
    private Optional<LockData> getLockData(LockId lockId) {
        List<LockData> locks = jdbcTemplate.query(
                "select * from locks where lockid = ?",
                lockDataRowMapper, lockId.getValue());
        return handleExpiration(locks);
    }

    @Transactional
    @Override
    // LockId에 해당하는 잠금 유효 시간을 inc만큼 늘린다.
    public void extendLockExpiration(LockId lockId, long inc) throws LockException {
        Optional<LockData> lockDataOpt = getLockData(lockId);
        LockData lockData =
                lockDataOpt.orElseThrow(() -> new NoLockException());
        jdbcTemplate.update(
                "update locks set expiration_time = ? where type = ? AND id = ?",
                new Timestamp(lockData.getTimestamp() + inc),
                lockData.getType(), lockData.getId());
    }

    @Transactional
    @Override
    // LockId에 해당하는 lock 삭제 
    public void releaseLock(LockId lockId) throws LockException {
        jdbcTemplate.update("delete from locks where lockid = ?", lockId.getValue());
    }

    @Autowired
    public void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public void setLockTimeout(int lockTimeout) {
        this.lockTimeout = lockTimeout;
    }
}
```
