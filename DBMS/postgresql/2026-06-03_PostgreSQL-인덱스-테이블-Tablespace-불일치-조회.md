# PostgreSQL 인덱스·테이블 Tablespace 불일치 조회

- **태그**: #PostgreSQL #Tablespace #인덱스 #시스템카탈로그
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-index와 table 이 사용하는 tablespace 조회쿼리.md`

## 1. 핵심 요약
- 인덱스와 소속 테이블이 서로 **다른 tablespace**를 사용하는 경우만 골라내는 진단 쿼리 — tablespace 배치 이상 탐지에 활용한다.

## 2. 상세 설명

### 배경
PostgreSQL에서 테이블과 인덱스는 각각 독립적인 tablespace에 배치될 수 있다. 의도적인 경우도 있지만, 실수로 분리된 경우 I/O 계획이 엇나가거나 유지보수 스크립트가 오작동할 수 있다. 이 쿼리는 불일치 쌍을 한눈에 찾아준다.

### 쿼리

```sql
SELECT
    i.relname   AS index_name,
    tsi.spcname AS index_tbsp,
    t.relname   AS table_name,
    tst.spcname AS table_tbsp
FROM pg_class t                          /* 테이블 */
JOIN pg_tablespace tst
  ON (t.reltablespace = tst.oid
      OR (t.reltablespace = 0 AND tst.spcname = 'pg_default'))
JOIN pg_index pgi
  ON pgi.indrelid = t.oid
JOIN pg_class i                          /* 인덱스 */
  ON pgi.indexrelid = i.oid
JOIN pg_tablespace tsi
  ON (i.reltablespace = tsi.oid
      OR (i.reltablespace = 0 AND tsi.spcname = 'pg_default'))
WHERE i.relname NOT LIKE 'pg_toast%'
  AND i.reltablespace != t.reltablespace;
```

### 출력 예시
```
   index_name   | index_tbsp | table_name |  table_tbsp
----------------+------------+------------+--------------
 customer_pkey  | pg_default | customer   | new_cust_tbs
(1개 행)
```

### 핵심 로직 설명

| 조인 조건 | 의미 |
|-----------|------|
| `t.reltablespace = tst.oid` | 테이블에 tablespace가 명시된 경우 |
| `t.reltablespace = 0 AND tst.spcname = 'pg_default'` | tablespace 미지정(0) → pg_default로 매핑 |
| `AND i.reltablespace != t.reltablespace` | 인덱스·테이블 tablespace OID가 다른 경우만 선택 |

- `reltablespace = 0`은 DB 기본 tablespace(보통 `pg_default`) 상속을 의미하므로, 단순 OID 비교만으로는 놓칠 수 있다. 이 쿼리는 0을 pg_default로 명시 매핑하여 정확하게 비교한다.
- `pg_toast%` 인덱스는 내부 TOAST 관리용이므로 필터링한다.

### 활용 시나리오
1. **tablespace 재배치 작업 전후 검증** — `ALTER INDEX ... SET TABLESPACE` 누락 확인
2. **스토리지 감사** — 특정 볼륨(tablespace)에 배치된 객체 현황 점검
3. **DBA 체크리스트** — 신규 DB 구성 후 배치 이상 여부 확인

## 3. 연관 개념 (지식 연결)
- [[2026-06-03_PostgreSQL-Tablespace-사용-객체-조회]]
- [[2026-06-03_PostgreSQL-인덱스-정보-조회]]
- [[2026-06-03_PostgreSQL-인덱스-상세정보-조회-di+]]
- [[2026-06-03_PostgreSQL-Tablespace-정보-조회]] — 모든 tablespace 이름과 파일시스템 경로 확인
