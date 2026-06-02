# PostgreSQL AutoVacuum 설정 확인

- **태그**: #PostgreSQL #AutoVacuum #모니터링 #운영
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-테이블에 설정된 auto vacuum 설정 확인.md`

## 1. 핵심 요약
- PostgreSQL 테이블별 Auto Vacuum 설정은 `pg_class.reloptions`(개별 설정) · `pg_stat_user_tables`(실행 이력) · `pg_settings`(전역 기본값) 세 가지 경로로 확인한다.

## 2. 상세 설명

Auto Vacuum은 Dead Tuple 정리와 통계 갱신(ANALYZE)을 자동으로 수행하는 PostgreSQL 백그라운드 프로세스다. 설정은 전역(postgresql.conf)과 테이블별(Storage Parameters) 두 계층으로 관리된다.

---

### 2-1. 특정 테이블의 개별 설정 확인

`ALTER TABLE … SET` 또는 테이블 생성 시 직접 부여한 파라미터를 확인한다.

#### SQL 쿼리
```sql
SELECT relname, reloptions
FROM pg_class
WHERE relname = '테이블명';
```
- `reloptions` 컬럼에 `autovacuum_enabled`, `autovacuum_vacuum_threshold` 등 테이블 전용 설정이 배열로 표시된다.
- 값이 `NULL`이면 전역 기본값(Global Default)을 따르고 있는 것이다.

#### psql 메타 명령어
```
\d+ 테이블명
```
- 출력 하단 **"Options"** 섹션에서 동일한 정보를 확인할 수 있다.

---

### 2-2. 모든 테이블의 Auto Vacuum 실행 이력 + 상태 확인

```sql
SELECT
    schemaname,
    relname,
    n_dead_tup,        -- 데드 튜플 수
    n_live_tup,        -- 활성 튜플 수
    last_autovacuum,   -- 마지막 자동 VACUUM 실행 시각
    last_autoanalyze   -- 마지막 자동 ANALYZE 실행 시각
FROM pg_stat_user_tables
WHERE relname = '테이블명';
```

---

### 2-3. 전역(시스템) 기본값 확인

테이블에 개별 설정이 없으면 `postgresql.conf`의 전역값이 적용된다.

```sql
SELECT name, setting, unit, short_desc
FROM pg_settings
WHERE name LIKE 'autovacuum%';
```

---

### 2-4. 설정 튜닝 팁

| 상황 | 권장 조치 |
|------|-----------|
| Dead Tuple이 많아 디스크 낭비·쿼리 속도 저하 | `autovacuum_vacuum_scale_factor` 낮게 설정 (예: 0.01) |
| 업데이트·삭제가 빈번한 대용량 테이블 | 해당 테이블에만 개별 파라미터를 더 공격적으로 설정 |
| autovacuum 비활성 테이블 존재 | `pg_class.reloptions`에서 `autovacuum_enabled=false` 여부 확인 |

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_PostgreSQL-통계정보-뷰-종류]]
- 관련 링크: [[2026-06-02_xmin과-xmax-개념]]
- 관련 링크: [[2026-06-03_PostgreSQL-시스템설정-Parameter-조회-변경]]
