# pg_stat_statements 활용

- **태그**: #PostgreSQL #쿼리모니터링 #성능분석 #통계확장
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-pg_stat_statements 활용.md`

## 1. 핵심 요약
- `pg_stat_statements` 확장은 PostgreSQL 서버에서 실행된 모든 SQL 문의 호출 횟수·총/평균/최소/최대 실행 시간을 누적 집계하여 슬로우 쿼리 원인 분석에 활용한다.

## 2. 상세 설명

`pg_stat_statements`는 PostgreSQL 확장 모듈로, 서버에서 실행된 SQL의 실행 통계를 동일한 뷰 이름(`public.pg_stat_statements`)으로 누적 제공한다. `shared_preload_libraries`에 등록하고 `CREATE EXTENSION`으로 활성화해야 사용할 수 있다.

### 주요 컬럼

| 컬럼 | 설명 |
|------|------|
| `query` | 정규화된 SQL 텍스트 |
| `calls` | 누적 호출 횟수 |
| `total_exec_time` | 누적 총 실행 시간 (ms) |
| `mean_exec_time` | 평균 실행 시간 (ms) |
| `stddev_exec_time` | 실행 시간 표준 편차 (ms) |
| `min_exec_time` | 최소 실행 시간 (ms) |
| `max_exec_time` | 최대 실행 시간 (ms) |
| `rows` | 누적 반환(영향) 행 수 |

### 총 실행 시간 상위 10개 쿼리 (ms 단위)

```sql
SELECT
    query,
    calls,
    total_exec_time AS total_time,
    rows,
    100.0 * total_exec_time / sum(total_exec_time) OVER () AS percent_total_time,
    mean_exec_time  AS mean_time,
    stddev_exec_time AS stddev_time,
    min_exec_time   AS min_time,
    max_exec_time   AS max_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

### 초 단위 + 한글 컬럼명 버전

```sql
SELECT
    query,
    calls                                                     AS "호출횟수",
    total_exec_time  / 1000                                   AS "총 소요시간",
    rows,
    100.0 * total_exec_time / sum(total_exec_time) OVER ()   AS "총 실행 시간의 백분율",
    mean_exec_time   / 1000                                   AS "소요된 평균 시간",
    stddev_exec_time / 1000                                   AS "실행 시간의 표준 편차",
    min_exec_time    / 1000                                   AS "소요된 최소 시간",
    max_exec_time    / 1000                                   AS "소요된 최대 시간"
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
```

### 유저명·DB명 포함 버전 (pg_database, pg_roles 조인)

```sql
SELECT
    rolname  AS username,
    datname  AS database_name,
    query,
    calls                                                     AS "호출횟수",
    total_exec_time  / 1000                                   AS "총 소요시간",
    rows,
    100.0 * total_exec_time / sum(total_exec_time) OVER ()   AS "총 실행 시간의 백분율",
    mean_exec_time   / 1000                                   AS "소요된 평균 시간",
    stddev_exec_time / 1000                                   AS "실행 시간의 표준 편차",
    min_exec_time    / 1000                                   AS "소요된 최소 시간",
    max_exec_time    / 1000                                   AS "소요된 최대 시간"
FROM pg_stat_statements
JOIN pg_database ON pg_stat_statements.dbid  = pg_database.oid
JOIN pg_roles    ON pg_stat_statements.userid = pg_roles.oid
ORDER BY total_exec_time DESC
LIMIT 10;
```

### 최대 수행시간 상위 50개 쿼리 (pg_user, pg_stat_database 조인)

```sql
SELECT
    a.userid,
    b.usename,
    a.dbid,
    c.datname,
    a.queryid,
    substr(a.query, 1, 100) AS query,
    a.calls,
    a.total_exec_time,
    a.min_exec_time,
    a.max_exec_time,
    a.rows
FROM public.pg_stat_statements a
JOIN pg_catalog.pg_user         b ON a.userid = b.usesysid
JOIN pg_catalog.pg_stat_database c ON a.dbid  = c.datid
ORDER BY a.max_exec_time DESC
LIMIT 50;
```

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_PostgreSQL-통계정보-뷰-종류]]
- 관련 링크: [[2026-06-02_pg_stat_activity-실행중인쿼리-확인]]
- 관련 링크: [[2026-06-03_PostgreSQL-EXPLAIN-옵션]]
- 관련 링크: [[2026-06-03_PostgreSQL-Cache-Hit-Miss-비율-조회]]
- 관련 링크: [[2026-06-03_Oracle-v$sql-스키마별-쿼리수행통계]]
