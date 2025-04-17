# isolation level = 격리수준
여러 트랜잭션이 동시에 처리될 때, 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있게 허용할지 여부를 결정하는 것
## 1. Read Uncommitted 
- 다른 트랜잭션이 커밋하지 않은 데이터도 읽을 수 있음
- 성능이 가장 빠르나 데이터 무결성에 매우 취약함

### `Dirty Read` 발생
- 아직 커밋되지 않은 값을 읽음
- insert 후에 commit하지 않았지만 insert한 데이터가 다른 트랜잭션에서도 읽힘

```mysql
  -- 트랜잭션 A
  start transaction;
  insert into users(name) values("홍길동");

  -- 트랜잭션 B
  start transaction;
  select * from users;
```

## 2. Read Committed
- 커밋한 데이터만 읽을 수 있음
- Dirty read 방지

```mysql
  -- 트랜잭션 A
  start transaction;
  insert into users(name) values("홍길동");

  -- 트랜잭션 B
  start transaction;
  select * from users;

  -- 트랜잭션 A
  commit;

  -- 트랜잭션 B
  select * from users; -- 데이터를 읽을 수 있음
```

- 트랜잭션A에서 insert한 데이터를 언두로그에 기록해 둠
  > 언두로그 Undo Log : 트랜잭션의 일관성과 롤백을 보장하기 위해 사용하는 로그
  - 트랜잭션 A에서 데이터를 변경하다가 롤백하면, 이전 값으로 되돌려야 하므로 필요

###  Non-Repeatable Read 발생
- 한 트랜잭션 내에서 읽은 결과가 달라지는 것

```mysql
    -- 트랜잭션 A (T1)
    SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
    START TRANSACTION;
    SELECT name FROM users WHERE id = 1;
    -- 결과: "홍길동"
    
    -- 트랜잭션 B (T2)
    START TRANSACTION;
    UPDATE users SET name = "이몽룡" WHERE id = 1;
    COMMIT;
    
    -- 트랜잭션 A (T1) 계속
    SELECT name FROM users WHERE id = 1;
    -- 결과: "이몽룡" ← 값이 바뀜!
```

## 3. Repeatable Read
- 트랜잭션 내에서 같은 SELECT는 항상 같은 결과를 반환
- MySQL의 기본 격리 수준

```sql   
    -- 트랜잭션 A
    START TRANSACTION;
    SELECT * FROM users WHERE id = 1;
    -- 결과: name = '홍길동'
    
    -- 트랜잭션 B
    START TRANSACTION;
    UPDATE users SET name = '이몽룡' WHERE id = 1;
    COMMIT;
    
    -- 트랜잭션 A (계속)
    SELECT * FROM users WHERE id = 1;
    -- 결과: name = '홍길동' (변화 없음!)
```
- 트랜잭션 A는 처음 쿼리를 실행했을 때의 스냅샷 버전을 기준으로 계속 데이터를 읽음
  - 트랜잭션 B가 데이터를 바꾸고 커밋하더라도 트랜잭션 A는 Undo Log에 저장된 과거 데이터를 읽기 때문에 조회 결과가 항상 동일

### Phantom Read : 조건에 부합하는 row 수가 바뀜
```sql
      -- 트랜잭션 A
      START TRANSACTION;
      SELECT * FROM users WHERE age > 20;
      -- 결과: 5명
    
      -- 트랜잭션 B
      START TRANSACTION;
      INSERT INTO users(name, age) VALUES('임꺽정', 25);
      COMMIT;
    
      -- 트랜잭션 A
      SELECT * FROM users WHERE age > 20;
      -- 결과: 6명 ← 팬텀 리드 발생
```
- 새로운 row가 조건을 만족하기 때문에 생기는 현상
- 읽은 데이터는 같지만 존재하는 행의 개수가 달라지는 것 <br>
-> innoDB에서는 팬텀 리드를 Gap Lock이라는 특별한 락을 사용해서 삽입을 막아버림
  

## 4. Serializable
- 트랜잭션을 순차적으로 실행하는 것처럼 동작
- 락 경쟁, 성능 저하가 발생할 수 있음
```sql
-- 트랜잭션 A
  SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
  START TRANSACTION;
  SELECT * FROM users WHERE age > 20;
  -- 결과: 5명
      
  -- 트랜잭션 B
  START TRANSACTION;
  INSERT INTO users(name, age) VALUES('임꺽정', 25);
  -- ❌ 대기 or 에러 (트랜잭션 A가 종료되기 전까지 삽입 불가!)
      
  -- 트랜잭션 A (계속)
  SELECT * FROM users WHERE age > 20;
  -- 결과: 여전히 5명 (팬텀 리드 없음)
  ```

- 모든 SELECT문에도 공유 락을 걸고 해당 조건 범위(Gap)에 삽입도 막는 방식을 사용함
- 다른 트랜잭션이 해당 범위에 write를 시도하면 대기하거나 즉시 에러 발생
- 실제로는 읽기조차도 락을 걸기 때문에 모든 트랜잭션이 순차적으로 실행되는 것과 거의 비슷함

## ex) 이벤트 선착순 참여 개발
- 동시에 수천 명이 이벤트 버튼을 눌러도 정확히 100명만 참여
- 중복 참여 방지, 데이터 꼬임 방지 <br>
-> Read Committed 기반 + Redis 캐시 사용

```typescript
 @Transactional('READ COMMITTED')
async participate(userId: string, eventId: string) {
    const redis = this.redisService.getClient();

    // 1. Redis로 중복 참여 방지
    const isAlready = await redis.get(`event:${userId}`);
    if (isAlready) throw new Error('이미 참여하셨습니다');

    await redis.set(`event:${userId}`, true, 'EX', 60); // 임시 락 (60초 유효)

    // 2. DB 트랜잭션 시작 (READ_COMMITTED로)
    const eventUsers = await this.eventRepository
        .createQueryBuilder('event_participants')
        .where('event_id = :eventId', { eventId })
        .getCount();

    if (eventUsers >= 100) {
        throw new Error('마감되었습니다');
    }

    // 3. 참여 등록
    await this.eventRepository.save({
        eventId,
        userId,
    });
}
```


### `Repeatable Read`보다 `Read Committed`를 사용
- Repeatable Read는 더 많은 잠금을 유발해 락 경쟁과 지연 발생
- Read Committed는 커밋된 데이터만 보지만 락 경쟁이 적어 동시성에 유리
- 대신 FOR UPDATE 같은 명시적인 락으로 정합성을 확보

