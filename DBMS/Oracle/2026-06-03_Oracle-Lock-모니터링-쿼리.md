# Oracle Lock 모니터링 쿼리

- **태그**: #Oracle #Lock #모니터링 #DBA
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-lock 모니터링 쿼리.md`

## 1. 핵심 요약
- `dba_lock` 뷰와 `v$lock`/`v$session` 뷰를 이용해 Oracle 세션의 TX·TM 락 대기 현황과 블로킹 세션을 실시간으로 진단하는 쿼리 모음

## 2. 상세 설명

### 전체 Lock 대기 상황 모니터링 (`dba_lock`)

TX(트랜잭션 락)와 TM(DML 락)을 한눈에 조회하며, 블로킹 중인 세션에 `<<<<<` 표시를 붙여 즉시 식별할 수 있다.

- `lock_type = 'Transaction'` → TX 락: `lock_id1`에서 USN(Undo Segment Number)과 SLOT을 비트 연산으로 분리
- `lock_type = 'DML'` → TM 락: `lock_id1`로 `dba_objects`를 조회해 대상 테이블명 표시
- `blocking_others = 'Blocking'` → 다른 세션을 블로킹 중인 세션 식별

```sql
select l.session_id SID
     ,(case when lock_type = 'Transaction' then 'TX' 
            when lock_type = 'DML' then 'TM' end) TYPE
     , mode_held
     , mode_requested mode_reqd
     ,(case when lock_type = 'Transaction' then 
                 to_char(trunc(lock_id1/power(2,16)))
            when lock_type = 'DML' then 
                (select object_name from dba_objects 
                 where object_id = l.lock_id1)
       end) "USN/Table"
     ,(case when lock_type = 'Transaction' then 
                 bitand(lock_id1, to_number('ffff', 'xxxx')) + 0
       end) "SLOT"
     ,(case when lock_type = 'Transaction' then 
                to_number(lock_id2) end) "SQN"
     ,(case when blocking_others = 'Blocking' then ' <<<<<' end) Blocking
from   dba_lock l
where  lock_type in ('Transaction', 'DML' )
order by session_id, lock_type, lock_id1, lock_id2
```

### 현재 유저 Lock 상태 조회 (`v$lock` + `v$session`)

현재 접속한 유저(`USER`)가 보유하거나 요청 중인 락을 `v$lock`과 `v$session`을 조인해 조회한다.

- `lmode`: 현재 보유 중인 락 모드 (0=없음, 2=Row-S, 3=Row-X, 6=Exclusive 등)
- `request`: 요청 중인 락 모드 (0이 아니면 대기 중)
- `block`: 1이면 다른 세션을 블로킹 중

```sql
select username, v$lock.sid, id1, id2
           , lmode, request, block, v$lock.type
from  v$lock, v$session
where v$lock.sid = v$session.sid
 and  v$session.username = USER
```

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_블로킹-쿼리-관계-조회]]
- 관련 링크: [[2026-06-02_RockyLinux9.7-Oracle19c-설치]]
