# PostgreSQL 대량 랜덤 데이터 삽입 — generate_series + random()

- **태그**: #PostgreSQL #테스트데이터 #generate_series #랜덤데이터
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-테이블에 대량의 랜덤 데이터를 삽입하는 쿼리.md`

## 1. 핵심 요약
- `generate_series()`와 `random()`을 SELECT 절에서 조합하여 단일 INSERT 문으로 테이블에 대량의 랜덤 테스트 데이터를 삽입하는 순수 SQL 방법

## 2. 상세 설명

### 기본 패턴

```sql
INSERT INTO p_sales (product_id, total_quantity, sale_date)
SELECT
    (random() * 100)::int + 1 AS product_id,
    (random() * 1000)::int + 1 AS total_quantity,
    generate_series('2023-01-01'::date, '2023-12-31'::date, '1 day'::interval) AS sale_date
FROM generate_series(1, 10000);
```

### 핵심 함수 설명

| 함수 | 역할 | 예시 |
|------|------|------|
| `generate_series(1, N)` | 1~N의 정수 시퀀스 생성 → 행 수 결정 | `generate_series(1, 10000)` → 10,000행 |
| `generate_series(start, end, interval)` | 날짜 범위를 간격 단위로 생성 | `'2023-01-01'~'2023-12-31'` 1일 간격 |
| `random()` | 0.0~1.0 사이 유사난수 반환 | `(random() * 100)::int + 1` → 1~100 정수 |

### 랜덤 정수 생성 공식

```
(random() * N)::int + offset
```

- `random() * 100` → 0.0 ~ 99.xxx
- `::int` → 소수점 버림 → 0 ~ 99
- `+ 1` → 1 ~ 100

### 주의 사항

- **행 수 폭발 위험**: FROM 절의 `generate_series(1, 10000)` × SELECT 절의 날짜 시리즈(365일)가 **크로스 조인**되어 실제 삽입 건수는 10,000 × 365 = **3,650,000건**이 될 수 있습니다. 특정 건수만 맞추려면 날짜도 `random()`으로 생성해야 합니다.
- `random()`은 PostgreSQL 의사난수 함수입니다. 재현 가능한 결과가 필요하면 먼저 `SELECT setseed(0.5);`로 시드를 고정합니다.
- PL/pgSQL DO 블록 방식([[2026-06-03_PLpgSQL-날짜생성-판매데이터-삽입]])보다 코드가 간결하지만, 조인 폭발에 유의해야 합니다.

### 날짜를 랜덤으로 생성하는 안전한 변형

```sql
INSERT INTO p_sales (product_id, total_quantity, sale_date)
SELECT
    (random() * 100)::int + 1,
    (random() * 1000)::int + 1,
    '2023-01-01'::date + (random() * 364)::int
FROM generate_series(1, 10000);
```

이 방식은 FROM 절의 `generate_series(1, 10000)`만 사용하므로 정확히 10,000건이 삽입됩니다.

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-03_PLpgSQL-날짜생성-판매데이터-삽입]]
- 관련 링크: [[2026-06-02_PostgreSQL-Advanced-SQL-실습가이드]]
