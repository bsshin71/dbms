# PostgreSQL 파티션 부모·자식 테이블 계층 조회

- **태그**: #PostgreSQL #파티션 #시스템카탈로그
- **작성일**: 2026-06-02
- **참조 원본**: `raw/clippings/2026-06-02-파티션 테이블의 부모테이블과 자식테이블 조회.md`

## 1. 핵심 요약
- `partition_hierarchy` 뷰(또는 CTE)를 통해 PostgreSQL 파티션 테이블의 부모-자식 계층 구조를 한 쿼리로 조회한다.

## 2. 상세 설명

### 조회 쿼리

```sql
SELECT schema_name, table_name, COALESCE(parent_name, 'Root Partition') AS parent_name
FROM partition_hierarchy
ORDER BY schema_name, parent_name, table_name;
```

### 쿼리 구성 요소

| 요소 | 설명 |
|------|------|
| `schema_name` | 파티션이 속한 스키마 이름 |
| `table_name` | 파티션 테이블 이름 (부모 또는 자식) |
| `COALESCE(parent_name, 'Root Partition')` | `parent_name`이 NULL이면 최상위 루트 파티션으로 표시 |
| `FROM partition_hierarchy` | 파티션 계층 정보를 담은 뷰 또는 CTE |
| `ORDER BY schema_name, parent_name, table_name` | 스키마 → 부모 → 자식 순으로 정렬하여 계층 파악 용이 |

### `partition_hierarchy` 뷰 구성 (배경 지식)

PostgreSQL 시스템 카탈로그를 활용하면 아래와 같이 파티션 계층 뷰를 직접 정의할 수 있다.

```sql
CREATE VIEW partition_hierarchy AS
WITH RECURSIVE ph AS (
    -- 루트 파티션 (부모 없음)
    SELECT n.nspname AS schema_name,
           c.relname AS table_name,
           NULL::text AS parent_name
    FROM pg_class c
    JOIN pg_namespace n ON n.oid = c.relnamespace
    WHERE c.relkind = 'p'
      AND NOT EXISTS (SELECT 1 FROM pg_inherits WHERE inhrelid = c.oid)
    UNION ALL
    -- 자식 파티션
    SELECT n.nspname,
           c.relname,
           ph.table_name
    FROM pg_class c
    JOIN pg_namespace n ON n.oid = c.relnamespace
    JOIN pg_inherits i ON i.inhrelid = c.oid
    JOIN ph ON ph.table_name = (
        SELECT relname FROM pg_class WHERE oid = i.inhparent
    )
)
SELECT schema_name, table_name, parent_name FROM ph;
```

- **`pg_class`**: 테이블·뷰·파티션 등 모든 릴레이션 정보 (`relkind = 'p'`가 파티션 테이블)
- **`pg_inherits`**: 부모-자식 관계 저장 (`inhparent` → `inhrelid`)
- **재귀 CTE**: 루트 파티션부터 내려가며 전체 트리를 펼침

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_PostgreSQL-통계정보-뷰-종류]], [[2026-06-02_PostgreSQL-Advanced-SQL-실습가이드]], [[2026-06-02_pg_stat_activity-실행중인쿼리-확인]], [[2026-06-03_PostgreSQL-파티션-자식테이블-pg_inherits-조회]]
