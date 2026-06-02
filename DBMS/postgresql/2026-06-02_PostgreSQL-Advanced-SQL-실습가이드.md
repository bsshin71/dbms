# PostgreSQL Advanced SQL 실습 가이드

- **태그**: #PostgreSQL #SQL고급 #쿼리튜닝 #Oracle이행
- **작성일**: 2026-06-02
- **원본 PDF**: [Advanced_SQL_PG_20250325_v0.9.pdf](../../../raw/Advanced_SQL_PG_20250325_v0.9.pdf) (227p)
- **참조 원본**: `raw/Advanced_SQL_PG_20250325_v0.9_extracted.md`

## 1. 핵심 요약
- Oracle 기반 중급 Advanced SQL 과정을 PostgreSQL 환경에서 재현한 227페이지 실습 가이드 — SELECT 기초부터 분석 함수·계층 질의·정규식까지, Oracle 경험자 시점에서 PG 전용 구문과 실행 계획 차이를 비교 설명한다.

## 2. 목차 구성

| 장 | 주제 | 페이지 |
|----|------|--------|
| 1 | 기본적인 SELECT 명령문 작성 | p.6 |
| 2 | INDEX를 사용하는 SQL | p.31 |
| 3 | 날짜 함수 활용 | p.48 |
| 4 | TOP-n 질의 활용 | p.62 |
| 5 | 조인·서브쿼리 활용 | p.77 |
| 6 | WITH 절 (CTE) | p.122 |
| 7 | 그룹 함수 활용 | p.128 |
| 8 | 분석 함수 활용 | p.154 |
| 9 | 계층 질의 활용 | p.193 |
| 10 | 정규식 (Regular Expression) | p.218 |
| 부록 | 기본적인 PG 이행 가이드 | p.226 |

## 3. 주요 PG 전용 포인트 (Oracle 대비)

### psql 공통 설정
```sql
ALTER SYSTEM SET jit = off;   -- pg_hint_plan과 충돌 방지
\pset pager off
\timing on
\set PROMPT1 '  %R%x> '      -- %x : 트랜잭션 상태 표시
```

### LIKE와 인덱스 (INDEX를 사용하는 SQL)
- PG에서 일반 B-tree 인덱스는 `LIKE 'prefix%'` 패턴에 기본적으로 사용되지 않음
- 해결책: 인덱스 생성 시 `text_pattern_ops` 옵션 지정
  ```sql
  CREATE INDEX idx ON emp(ename text_pattern_ops);
  ```
- 예외: 데이터베이스 생성 시 `COLLATE C`를 지정하면 일반 인덱스도 LIKE에 사용 가능
- `~~` 연산자는 실행 계획에서 LIKE를 의미함

### 후방 LIKE (끝 문자 검색)
```sql
-- 함수 기반 인덱스로 역순 검색 구현
CREATE INDEX ename_fbi ON emp(REVERSE(ename));

-- pg_hint_plan으로 IndexScan 강제
/*+ IndexScan(emp) */
SELECT empno, ename FROM emp
 WHERE REVERSE(ename) LIKE 'TT%';  -- 원래 '%TT' 검색
```

### 힌트 사용 (pg_hint_plan 확장)
```sql
/*+ IndexScan(테이블명) */   -- Full Scan 대신 Index Scan 강제
/*+ SeqScan(테이블명) */     -- 반대로 Seq Scan 강제
```

### 문자열 비교 특성
- `ename > 'JONES'` — 알파벳 대소 비교 가능 (한글도 범위 비교 적용)
- `BETWEEN '라' AND '마'` — 한글 범위 조건도 동작

### 트랜잭션 제어
```sql
-- Oracle의 AUTOCOMMIT off 대신 명시적 BEGIN 권장
BEGIN;
-- DML 작업 ...
ROLLBACK;  -- 또는 COMMIT;
```

## 4. 장별 핵심 개념 요약

### 1장. 기본적인 SELECT
- 문자열 대소 비교, LIKE, 와일드카드
- `INITCAP()`, `UPPER()` 등 문자 함수
- NULL 처리, CASE 표현식

### 2장. INDEX를 사용하는 SQL
- B-tree 인덱스 구조와 LIKE 제약
- 함수 기반 인덱스(Function-Based Index)
- `text_pattern_ops`로 LIKE 인덱스 활성화
- pg_hint_plan으로 실행 계획 제어

### 5장. 조인·서브쿼리
- INNER JOIN, OUTER JOIN (LEFT/RIGHT/FULL)
- 스칼라 서브쿼리, 인라인 뷰, 상관 서브쿼리
- EXISTS vs IN 성능 비교

### 6장. WITH 절 (CTE)
- Common Table Expression — 재귀 CTE(`WITH RECURSIVE`) 포함
- Oracle의 `CONNECT BY` 계층 질의를 CTE로 대체하는 패턴

### 8장. 분석 함수
- `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`
- `LAG()`, `LEAD()`로 이전/다음 행 참조
- `SUM() OVER(PARTITION BY ... ORDER BY ...)` 누적 합계

### 9장. 계층 질의
- Oracle `CONNECT BY` → PG `WITH RECURSIVE`로 변환
- 재귀 CTE 패턴: 앵커 쿼리 + 재귀 쿼리 + UNION ALL

### 10장. 정규식
- `~` (POSIX 정규식 일치), `~*` (대소문자 무시)
- `regexp_replace()`, `regexp_matches()`, `regexp_split_to_table()`

## 5. 실습 환경
- 데이터: HR 스키마(EMPLOYEES, DEPARTMENTS 등) + EMP/DEPT (Oracle 클래식 샘플)
- 확장: `pg_hint_plan` (실행 계획 힌트)
- 폰트: D2Coding (고정폭, SQL 가독성용)

## 6. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_블로킹-쿼리-관계-조회]], [[2026-06-02_xmin과-xmax-개념]], [[2026-06-01_MinTool4PG-설치법]], [[2026-06-02_pgvector-설치]]
