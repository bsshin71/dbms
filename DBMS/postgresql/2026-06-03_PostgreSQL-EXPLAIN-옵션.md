# PostgreSQL EXPLAIN 옵션

- **태그**: #PostgreSQL #쿼리튜닝 #실행계획
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-PG explain 옵션.md`

## 1. 핵심 요약
- `EXPLAIN` 계열 명령어로 PostgreSQL이 쿼리를 어떻게 실행하는지 단계별 분석 정보를 얻을 수 있으며, 옵션을 추가할수록 버퍼·시간·컬럼 상세 정보까지 볼 수 있다.

## 2. 상세 설명

| 명령어 | 실행 여부 | 주요 정보 |
|--------|-----------|-----------|
| `EXPLAIN` | 실행 안 함 | 옵티마이저의 **예상** 실행 계획(비용·행 수·폭) |
| `EXPLAIN (ANALYZE)` | 실행함 | 예상 vs **실제** 소요 시간, 실제 처리 행 수 |
| `EXPLAIN (ANALYZE, BUFFERS)` | 실행함 | ANALYZE 정보 + 각 노드별 **버퍼(캐시) hit/miss** 횟수 |
| `EXPLAIN (ANALYZE, BUFFERS, VERBOSE)` | 실행함 | 위 모두 + 각 노드의 **출력 컬럼 목록** + 내부 동작 깊이 분석 |

### 옵션별 활용 시나리오

- **EXPLAIN** — 쿼리를 실행하지 않아 DML에 안전. 인덱스 사용 여부, 조인 방식 등 빠른 1차 확인에 사용.
- **EXPLAIN (ANALYZE)** — 실제 실행 시간이 필요할 때. `SELECT`는 부작용 없음. `INSERT/UPDATE/DELETE`는 트랜잭션으로 감싸고 ROLLBACK.
- **EXPLAIN (ANALYZE, BUFFERS)** — I/O 병목 진단. `shared hit`(공유 버퍼 캐시)이 낮고 `shared read`가 높으면 디스크 읽기 과다.
- **EXPLAIN (ANALYZE, BUFFERS, VERBOSE)** — 복잡한 뷰·서브쿼리 분석 시 각 노드가 반환하는 컬럼을 확인하여 불필요한 컬럼 스캔 여부 파악.

### 주의 사항
- `ANALYZE` 옵션은 실제로 쿼리를 실행하므로 DML 시 **반드시 트랜잭션으로 감싸고 ROLLBACK** 해야 데이터가 변경되지 않는다.
- `BUFFERS` 옵션은 `ANALYZE` 없이 단독 사용할 수 없다.

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_PostgreSQL-통계정보-뷰-종류]]
- 관련 링크: [[2026-06-02_pg_stat_activity-실행중인쿼리-확인]]
- 관련 링크: [[2026-06-02_블로킹-쿼리-관계-조회]]
