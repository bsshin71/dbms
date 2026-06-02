# PostgreSQL Tablespace 사용 객체 조회

- **태그**: #PostgreSQL #tablespace #pg_class #시스템카탈로그
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-tabespace 를 사용하는 객체 조회.md`

## 1. 핵심 요약
- 특정 테이블스페이스(tablespace)에 배치된 모든 DB 객체(테이블·인덱스 등)를 `pg_class`와 `pg_tablespace` 시스템 카탈로그 조인으로 조회하는 쿼리

## 2. 상세 설명

### 조회 쿼리

```sql
SELECT
    c.relname                    AS object_name,
    c.relkind                    AS object_type,
    c.relfilenode                AS file_number,
    COALESCE(
        pg_tablespace_location(c.reltablespace),
        'default or database path'
    )                            AS tablespace_location
FROM pg_class c
JOIN pg_database d ON d.datname = current_database()
WHERE
    c.reltablespace = (SELECT oid FROM pg_tablespace WHERE spcname = 'sales_tbs')
    OR
    (c.reltablespace = 0
     AND d.dattablespace = (SELECT oid FROM pg_tablespace WHERE spcname = 'sales_tbs'));
```

### 핵심 원리

| 조건 | 의미 |
|------|------|
| `c.reltablespace = <oid>` | 객체가 명시적으로 해당 tablespace에 지정된 경우 |
| `c.reltablespace = 0` | 객체가 DB 기본 tablespace를 따르는 경우 |
| `d.dattablespace = <oid>` | DB 기본 tablespace가 해당 tablespace인 경우 |

두 조건을 OR로 결합하면 **명시 지정된 객체 + DB 기본 tablespace를 상속받은 객체** 모두를 빠짐없이 조회할 수 있다.

### 주요 컬럼 설명

- **`relkind`** — 객체 종류 코드: `r`=일반 테이블, `i`=인덱스, `S`=시퀀스, `v`=뷰, `m`=materialized view, `f`=외부 테이블
- **`relfilenode`** — 해당 객체의 데이터 파일 번호 (OS 파일 추적 시 활용)
- **`pg_tablespace_location()`** — tablespace의 실제 파일시스템 경로 반환; `pg_default`/`pg_global`은 NULL 반환

### 실행 결과 예시

```
 object_name | object_type | file_number |         tablespace_location
-------------+-------------+-------------+---------------------------------------------
 sales       | r           |       25297 | /home/postgres/pg18/data/user_tbs/sales_tbs
(1 row)
```

`sales_tbs` tablespace에 `sales` 테이블(relkind=`r`) 하나가 배치되어 있음을 확인.

### 활용 시나리오

- 특정 tablespace 사용 현황 파악 (용량 관리·마이그레이션 전 점검)
- tablespace 삭제 전 의존 객체 식별 (`DROP TABLESPACE` 실패 원인 분석)
- 파일시스템 경로 기반의 물리적 오브젝트 추적

## 3. 연관 개념 (지식 연결)
- [[2026-06-03_PostgreSQL-DB용량-확인-쿼리]] — tablespace별 용량 확인 쿼리와 병행 활용
- [[2026-06-02_PostgreSQL-전체-제약조건-조회]] — pg_class 시스템 카탈로그 활용 패턴
- [[2026-06-02_PostgreSQL-파티션-계층-조회]] — pg_class 기반 객체 계층 조회
- [[2026-06-03_PostgreSQL-인덱스-테이블-Tablespace-불일치-조회]] — 인덱스·테이블 간 tablespace 불일치 진단 쿼리
- [[2026-06-03_PostgreSQL-Tablespace-정보-조회]] — tablespace 이름과 파일시스템 경로 전체 목록 조회
