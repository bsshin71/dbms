# PostgreSQL 인덱스 상세 정보 조회 (psql \di+ 동등 쿼리)

- **태그**: #PostgreSQL #인덱스 #시스템카탈로그 #pg_class
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-현재 데이터베이스의 인덱스 상세 정보 조회를 위한 SQL 쿼리.md`

## 1. 핵심 요약
- `psql`의 `\di+` 명령어와 동일하게 `pg_catalog.pg_class`·`pg_namespace`·`pg_am` 등을 직접 조인하여 현재 데이터베이스의 모든 인덱스 상세 정보(액세스 메서드, 크기, 지속성, 소유자 등)를 조회하는 SQL 쿼리

## 2. 상세 설명

### 쿼리 목적

`pg_indexes` 뷰가 DDL 문자열 위주의 기본 정보를 제공하는 것과 달리, 이 쿼리는 `\di+`와 동일하게 **액세스 메서드(B-tree, Hash, GIN, GiST 등)**, **인덱스 크기**, **지속성(permanent/temporary/unlogged)**, **소유자**, **연결 테이블명** 등의 운영 상세 정보를 함께 반환합니다.

### SQL 쿼리

```sql
SELECT n.nspname                                        AS "Schema",
       c.relname                                        AS "Name",
       CASE c.relkind
           WHEN 'r' THEN 'table'
           WHEN 'v' THEN 'view'
           WHEN 'm' THEN 'materialized view'
           WHEN 'i' THEN 'index'
           WHEN 'S' THEN 'sequence'
           WHEN 't' THEN 'TOAST table'
           WHEN 'f' THEN 'foreign table'
           WHEN 'p' THEN 'partitioned table'
           WHEN 'I' THEN 'partitioned index'
       END                                              AS "Type",
       pg_catalog.pg_get_userbyid(c.relowner)          AS "Owner",
       c2.relname                                       AS "Table",
       CASE c.relpersistence
           WHEN 'p' THEN 'permanent'
           WHEN 't' THEN 'temporary'
           WHEN 'u' THEN 'unlogged'
       END                                              AS "Persistence",
       am.amname                                        AS "Access method",
       pg_catalog.pg_size_pretty(
           pg_catalog.pg_table_size(c.oid))             AS "Size",
       pg_catalog.obj_description(c.oid, 'pg_class')   AS "Description"
FROM   pg_catalog.pg_class c
LEFT JOIN pg_catalog.pg_namespace n  ON n.oid = c.relnamespace
LEFT JOIN pg_catalog.pg_am am        ON am.oid = c.relam
LEFT JOIN pg_catalog.pg_index i      ON i.indexrelid = c.oid
LEFT JOIN pg_catalog.pg_class c2     ON i.indrelid = c2.oid
WHERE  c.relkind IN ('i', 'I', '')
  AND  n.nspname <> 'pg_catalog'
  AND  n.nspname !~ '^pg_toast'
  AND  n.nspname <> 'information_schema'
  AND  pg_catalog.pg_table_is_visible(c.oid)
ORDER BY 1, 2;
```

### 반환 컬럼 설명

| 컬럼 | 설명 |
|------|------|
| `Schema` | 인덱스가 속한 스키마 |
| `Name` | 인덱스 이름 |
| `Type` | 오브젝트 타입(`index` / `partitioned index`) |
| `Owner` | 인덱스 소유자 |
| `Table` | 인덱스가 걸린 테이블 이름 |
| `Persistence` | `permanent` / `temporary` / `unlogged` |
| `Access method` | 인덱스 알고리즘(btree, hash, gin, gist, brin 등) |
| `Size` | 인덱스 크기(사람이 읽기 쉬운 단위) |
| `Description` | `COMMENT ON INDEX ...`로 등록한 설명 |

### 필터링 조건

- `c.relkind IN ('i', 'I', '')` — 일반 인덱스와 파티션 인덱스만 대상
- 시스템 카탈로그(`pg_catalog`), TOAST 테이블(`pg_toast`), `information_schema` 제외
- `pg_table_is_visible(c.oid)` — 현재 `search_path`에서 가시적인 인덱스만 반환

### pg_indexes와의 차이

| 항목 | `pg_indexes` 뷰 | 이 쿼리(`\di+` 동등) |
|------|-----------------|----------------------|
| 액세스 메서드 | 미제공 | 제공(`am.amname`) |
| 인덱스 크기 | 미제공 | 제공(`pg_table_size`) |
| 지속성 | 미제공 | 제공(`relpersistence`) |
| DDL 문자열 | 제공(`indexdef`) | 미제공 |
| 파티션 인덱스 | 미포함 | 포함(`relkind='I'`) |

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-03_PostgreSQL-인덱스-정보-조회]]
- 관련 링크: [[2026-06-02_PostgreSQL-전체-제약조건-조회]]
- 관련 링크: [[2026-06-02_PostgreSQL-통계정보-뷰-종류]]
- 관련 링크: [[2026-06-03_PostgreSQL-psql-주요명령어]]
