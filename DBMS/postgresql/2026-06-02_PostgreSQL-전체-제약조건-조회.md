# PostgreSQL 전체 제약조건 조회

- **태그**: #PostgreSQL #시스템카탈로그 #제약조건 #메타데이터
- **작성일**: 2026-06-02
- **참조 원본**: `raw/clippings/2026-06-02-현재 데이터베이스의 모든  제약조건 확인.md`

## 1. 핵심 요약
- `pg_constraint`, `pg_attribute` 시스템 카탈로그를 조인하여 현재 DB의 모든 테이블에 정의된 PRIMARY KEY·UNIQUE·FOREIGN KEY·CHECK·NOT NULL 제약조건을 한 번에 조회하는 쿼리.

## 2. 상세 설명

### 사용 시스템 카탈로그

| 테이블 | 역할 |
|--------|------|
| `pg_class` | 테이블·인덱스 등 릴레이션 목록 |
| `pg_namespace` | 스키마(네임스페이스) 정보 |
| `pg_attribute` | 컬럼 정보 (attnotnull → NOT NULL 여부 포함) |
| `pg_constraint` | PRIMARY KEY·UNIQUE·FOREIGN KEY·CHECK 제약조건 |

### 조회 쿼리

```sql
SELECT n.nspname AS table_schema,
       c.relname AS table_name,
       CASE
           WHEN con.conname IS NOT NULL THEN con.conname
           ELSE c.relname || '_' || a.attname || '_not_null'
       END AS constraint_name,
       CASE
           WHEN con.contype = 'p' THEN 'PRIMARY KEY'
           WHEN con.contype = 'u' THEN 'UNIQUE'
           WHEN con.contype = 'f' THEN 'FOREIGN KEY'
           WHEN con.contype = 'c' THEN 'CHECK'
           WHEN a.attnotnull THEN 'NOT NULL'
       END AS constraint_type,
       a.attname AS column_name
FROM pg_class c
JOIN pg_namespace n ON n.oid = c.relnamespace
JOIN pg_attribute a ON a.attrelid = c.oid
LEFT JOIN pg_constraint con
       ON con.conrelid = c.oid
      AND (con.conkey @> array[a.attnum] OR con.conkey IS NULL)
WHERE n.nspname NOT IN ('pg_catalog', 'information_schema')
  AND c.relkind = 'r'
  AND (con.contype IS NOT NULL OR a.attnotnull)
  AND a.attname NOT IN ('cmax', 'cmin', 'ctid', 'tableoid', 'xmax', 'xmin')
ORDER BY table_schema, table_name, constraint_name;
```

### 주요 포인트

- **`pg_constraint.contype`**: `'p'`=PRIMARY KEY, `'u'`=UNIQUE, `'f'`=FOREIGN KEY, `'c'`=CHECK
- **NOT NULL 처리**: `pg_constraint`에는 NOT NULL이 없으므로 `pg_attribute.attnotnull`로 별도 감지하고, 이름은 `{테이블명}_{컬럼명}_not_null` 패턴으로 생성
- **`conkey @> array[a.attnum]`**: 제약조건이 해당 컬럼을 포함하는지 배열 포함 연산자(`@>`)로 확인
- **시스템 컬럼 제외**: `cmax`, `cmin`, `ctid`, `tableoid`, `xmax`, `xmin`은 PostgreSQL 내부 시스템 컬럼이므로 필터링

### 출력 예시

```
table_schema | table_name  | constraint_name                  | constraint_type | column_name
-------------+-------------+----------------------------------+-----------------+-------------
mkt          | customer    | customer_email_not_null          | NOT NULL        | email
mkt          | customer    | customer_first_name_not_null     | NOT NULL        | first_name
mkt          | customer    | customer_pkey                    | PRIMARY KEY     | customer_id
mkt          | new_sales   | new_sales_order_date_not_null    | NOT NULL        | order_date
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-02_pg_stat_activity-실행중인쿼리-확인]]
- [[2026-06-02_블로킹-쿼리-관계-조회]]
- [[2026-06-02_PostgreSQL-통계정보-뷰-종류]]
- [[2026-06-02_xmin과-xmax-개념]]
- [[2026-06-03_PostgreSQL-인덱스-정보-조회]]
