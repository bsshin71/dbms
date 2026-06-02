# PostgreSQL 파티션 자식 테이블 직접 조회 (pg_inherits)

- **태그**: #PostgreSQL #파티션 #pg_inherits #시스템카탈로그
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-파티션 테이블 관계 조회 쿼리.md`

## 1. 핵심 요약
- `pg_catalog.pg_inherits`를 직접 조회하여 특정 파티션 테이블의 직계 자식 파티션 목록을 한 줄 쿼리로 빠르게 확인한다.

## 2. 상세 설명

### 조회 쿼리

```sql
SELECT inhrelid::regclass AS child
FROM pg_catalog.pg_inherits
WHERE inhparent = 'p_sales'::regclass;
```

### 쿼리 구성 요소

| 요소 | 설명 |
|------|------|
| `inhrelid::regclass` | 자식 파티션의 OID를 테이블 이름(regclass)으로 캐스팅 |
| `pg_catalog.pg_inherits` | 부모-자식 상속 관계를 저장하는 시스템 카탈로그 |
| `inhparent = 'p_sales'::regclass` | 조회할 부모 파티션 테이블 이름을 OID로 변환하여 필터링 |

### 실행 결과 예시

```
child
-----------------
p_sales_2023_q1
p_sales_2023_q2
p_sales_2023_q3
p_sales_2023_q4
(4 rows)
```

### 재귀 CTE 방식과의 차이

| 방식 | 특징 |
|------|------|
| `pg_inherits` 직접 조회 (이 노트) | 단일 부모의 **직계 자식만** 반환, 쿼리 간결 |
| `partition_hierarchy` 뷰 (재귀 CTE) | 전체 다단계 계층을 트리 형태로 반환, 복잡한 계층에 적합 |

단순히 특정 파티션의 자식 목록만 확인할 때는 이 직접 조회 방식이 더 빠르고 간결하다.

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_PostgreSQL-파티션-계층-조회]], [[2026-06-02_PostgreSQL-전체-제약조건-조회]], [[2026-06-02_PostgreSQL-통계정보-뷰-종류]]
