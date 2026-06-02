# pg_stat_activity — 실행중인 쿼리 확인

- **태그**: #PostgreSQL #모니터링 #pg_stat_activity #세션관리
- **작성일**: 2026-06-02
- **참조 원본**: `raw/clippings/2026-06-02-실행중인 쿼리 확인.md`

## 1. 핵심 요약
- `pg_stat_activity` 뷰를 조회하면 현재 데이터베이스에서 실행 중인 모든 세션의 쿼리·상태·대기 이벤트를 실시간으로 파악할 수 있다.

## 2. 상세 설명

### pg_stat_activity 뷰
데이터베이스에서 현재 활성화된 세션 정보를 제공하는 시스템 뷰다. 실행 중인 쿼리, 세션 상태(`active` / `idle` 등), 대기 이벤트, 쿼리 시작 시각 등을 확인할 수 있다.

### 실행중인 쿼리 조회

```sql
SELECT pid, usename, query, state, query_start
FROM pg_stat_activity
WHERE state = 'active';
```

| 컬럼 | 설명 |
|------|------|
| `pid` | 백엔드 프로세스 ID |
| `usename` | 접속 사용자명 |
| `query` | 마지막으로 실행된(또는 현재 실행 중인) SQL |
| `state` | 세션 상태 (`active`, `idle`, `idle in transaction` 등) |
| `query_start` | 현재 쿼리 시작 시각 |

### 실습 예시

**터미널 1** — 60초 대기 쿼리 실행:
```sql
SELECT pg_sleep(60);
```

**터미널 2** — 실행 중 쿼리 확인:
```sql
SELECT pid, usename, query, state, query_start
FROM pg_stat_activity
WHERE state = 'active';
```

결과 예시:
```
 pid | usename  | query                      | state  | query_start
-----+----------+----------------------------+--------+-------------------------------
 102 | postgres | SELECT pid, usename, ...   | active | 2024-07-26 01:57:57.52554+00
 296 | postgres | select pg_sleep(60);       | active | 2024-07-26 01:57:54.447568+00
```

### pg_sleep() 함수
초 단위로 지정한 시간만큼 쿼리를 대기시키는 함수다. 장시간 실행되는 쿼리 모니터링 실습이나 잠금(Lock) 시나리오 재현에 활용한다.

```sql
SELECT pg_sleep(N);  -- N초 동안 대기
```

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_블로킹-쿼리-관계-조회]]
- 관련 링크: [[2026-06-02_xmin과-xmax-개념]]
- 관련 링크: [[2026-06-02_PostgreSQL-통계정보-뷰-종류]]
- 관련 링크: [[2026-06-03_PostgreSQL-EXPLAIN-옵션]]
- 관련 링크: [[2026-06-03_PostgreSQL-pg_stat_statements-활용]]
