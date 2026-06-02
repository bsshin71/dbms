# PostgreSQL 인덱스 정보 조회

- **태그**: #PostgreSQL #인덱스 #시스템카탈로그 #pg_indexes
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-현재데이터베이스의 index 정보정보 조회.md`

## 1. 핵심 요약
- `pg_indexes` 시스템 뷰를 조회하여 현재 데이터베이스에 정의된 모든 인덱스의 이름·소속 테이블·생성 구문을 한 번에 확인하는 방법

## 2. 상세 설명

### 기본 쿼리

```sql
SELECT schemaname, tablename, indexname, indexdef
FROM pg_indexes
WHERE schemaname <> 'pg_catalog';
```

### 반환 컬럼 설명

| 컬럼 | 설명 |
|------|------|
| `schemaname` | 인덱스가 속한 스키마 이름 |
| `tablename` | 인덱스가 걸린 테이블 이름 |
| `indexname` | 인덱스 이름 |
| `indexdef` | 인덱스 생성 DDL(`CREATE INDEX ...` 구문) |

### 조회 조건

- `schemaname <> 'pg_catalog'` 조건으로 PostgreSQL 내부 시스템 인덱스를 제외하고 사용자 정의 인덱스만 조회합니다.
- `public` 스키마뿐 아니라 다른 사용자 스키마의 인덱스도 함께 반환됩니다.

### 출력 예시

```
 schemaname | tablename | indexname  |                               indexdef
------------+-----------+------------+-----------------------------------------------------------------------
 public     | sales     | sales_pkey | CREATE UNIQUE INDEX sales_pkey ON public.sales USING btree (sales_id)
```

`indexdef`를 통해 인덱스 타입(btree, hash, gin, gist 등), 대상 컬럼, UNIQUE 여부를 한눈에 파악할 수 있습니다.

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-03_PostgreSQL-인덱스-상세정보-조회-di+]]
- 관련 링크: [[2026-06-02_PostgreSQL-전체-제약조건-조회]]
- 관련 링크: [[2026-06-02_PostgreSQL-통계정보-뷰-종류]]
- 관련 링크: [[2026-06-02_PostgreSQL-파티션-계층-조회]]
