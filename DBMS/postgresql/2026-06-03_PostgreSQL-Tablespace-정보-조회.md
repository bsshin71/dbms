# PostgreSQL Tablespace 정보 조회

- **태그**: #PostgreSQL #tablespace #pg_tablespace #시스템카탈로그
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-tablespace 정보 조회 쿼리.md`

## 1. 핵심 요약
- `pg_tablespace` 시스템 카탈로그와 `pg_tablespace_location()` 함수를 이용해 PostgreSQL 인스턴스에 정의된 모든 tablespace의 이름과 실제 파일시스템 경로를 조회하는 쿼리

## 2. 상세 설명

### 조회 쿼리

```sql
SELECT
    CASE
        WHEN pg_tablespace_location(oid) = '' AND spcname = 'pg_default'
            THEN current_setting('data_directory') || '/base/'
        WHEN pg_tablespace_location(oid) = '' AND spcname = 'pg_global'
            THEN current_setting('data_directory') || '/global/'
        ELSE pg_tablespace_location(oid)
    END AS spclocation,
    spcname
FROM pg_tablespace;
```

### 핵심 원리

`pg_tablespace_location()` 함수는 사용자 정의 tablespace의 경로를 반환하지만, 내장 tablespace(`pg_default`, `pg_global`)에 대해서는 빈 문자열을 반환한다. 이를 `CASE` 식으로 분기해 `data_directory` GUC 파라미터 값을 기반으로 실제 경로를 직접 조합한다.

| Tablespace | 경로 산출 방식 |
|------------|---------------|
| `pg_default` | `current_setting('data_directory') \|\| '/base/'` |
| `pg_global` | `current_setting('data_directory') \|\| '/global/'` |
| 사용자 정의 | `pg_tablespace_location(oid)` 직접 반환 |

### 실행 결과 예시

```
spclocation                                | spcname
-------------------------------------------+------------
/home/postgres/pg18/data/base/             | pg_default
/home/postgres/pg18/data/global/           | pg_global
/home/postgres/pg18/data/user_tbs/cust_tbs | cust_tbs
/home/postgres/pg18/data/user_tbs/sales_tbs| sales_tbs
```

### 활용 시나리오

- 현재 인스턴스에 존재하는 모든 tablespace와 물리 경로 일괄 확인
- 파일시스템 용량 모니터링·마이그레이션 계획 수립 시 기초 자료 수집
- `DROP TABLESPACE` 전 경로 확인 및 정리 대상 식별

## 3. 연관 개념 (지식 연결)
- [[2026-06-03_PostgreSQL-Tablespace-사용-객체-조회]] — 특정 tablespace에 배치된 객체 목록 조회
- [[2026-06-03_PostgreSQL-인덱스-테이블-Tablespace-불일치-조회]] — 인덱스·테이블 간 tablespace 불일치 진단
- [[2026-06-03_PostgreSQL-DB용량-확인-쿼리]] — tablespace별 용량 확인 쿼리와 병행 활용
