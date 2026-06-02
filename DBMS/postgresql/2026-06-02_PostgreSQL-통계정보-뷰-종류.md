# PostgreSQL 통계 정보 관련 뷰(View) 종류

- **태그**: #PostgreSQL #모니터링 #통계정보 #시스템뷰
- **작성일**: 2026-06-02
- **참조 원본**: `raw/clippings/2026-06-02-PostgreSQL 통계 정보관련 view 테이블 종류.md`

## 1. 핵심 요약
- PostgreSQL은 세션·테이블·인덱스·데이터베이스 단위의 운영 통계를 조회할 수 있는 내장 시스템 뷰를 제공하며, 이를 활용하면 성능 모니터링과 문제 진단을 수행할 수 있다.

## 2. 상세 설명

PostgreSQL의 통계 정보 관련 뷰는 크게 세션, 테이블, 인덱스, 데이터베이스 수준으로 구분된다.

| 뷰 이름 | 설명 |
|---------|------|
| `pg_stat_activity` | 데이터베이스 서버에서 실행 중인 모든 세션에 대한 정보를 제공 |
| `pg_stat_user_tables` | 시스템 테이블을 제외한 사용자 테이블에 대한 통계 정보를 제공 |
| `pg_stat_user_indexes` | 시스템 인덱스를 제외한 사용자 인덱스에 대한 통계 정보를 제공 |
| `pg_stat_database` | 각 데이터베이스 전체의 통계 정보를 제공 |

### pg_stat_activity
현재 접속된 세션의 PID, 사용자, 실행 중인 SQL, 상태(`active` / `idle` 등), 대기 이벤트, 쿼리 시작 시각 등을 실시간으로 조회한다. 블로킹 쿼리 진단 및 장시간 실행 쿼리 탐지에 가장 많이 활용된다.

### pg_stat_user_tables
테이블별로 순차 스캔 횟수(`seq_scan`), 인덱스 스캔 횟수(`idx_scan`), INSERT/UPDATE/DELETE 행 수, autovacuum 마지막 실행 시각 등을 제공한다. 불필요한 순차 스캔이 많은 테이블이나 vacuum 지연 여부를 파악할 때 유용하다.

### pg_stat_user_indexes
인덱스별로 스캔 횟수(`idx_scan`), 인덱스를 통해 반환된 행 수(`idx_tup_read` / `idx_tup_fetch`)를 제공한다. 사용되지 않는 인덱스를 발견하거나 인덱스 효율을 평가할 때 활용한다.

### pg_stat_database
데이터베이스별로 커밋·롤백 수, 블록 히트·읽기 비율(버퍼 캐시 효율), 교착상태 발생 횟수, 반환된 행 수 등 전체 부하 현황을 집계한다.

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_pg_stat_activity-실행중인쿼리-확인]]
- 관련 링크: [[2026-06-02_블로킹-쿼리-관계-조회]]
- 관련 링크: [[2026-06-02_PostgreSQL-전체-제약조건-조회]]
- 관련 링크: [[2026-06-03_PostgreSQL-인덱스-정보-조회]]
- 관련 링크: [[2026-06-03_PostgreSQL-DB용량-확인-쿼리]]
- 관련 링크: [[2026-06-03_PostgreSQL-psql-주요명령어]]
- 관련 링크: [[2026-06-03_PostgreSQL-AutoVacuum-설정-확인]]
- 관련 링크: [[2026-06-03_PostgreSQL-EXPLAIN-옵션]]
- 관련 링크: [[2026-06-03_PostgreSQL-Cache-Hit-Miss-비율-조회]]
- 관련 링크: [[2026-06-03_PostgreSQL-pg_stat_statements-활용]]
