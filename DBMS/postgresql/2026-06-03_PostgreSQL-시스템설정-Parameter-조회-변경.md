# PostgreSQL 시스템설정(Parameter) 조회 및 변경

- **태그**: #PostgreSQL #파라미터 #설정관리 #운영
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-시스템설정(Parameter) 조회 및 변경.md`

## 1. 핵심 요약
- PostgreSQL 서버의 시스템 설정값(Parameter)을 psql 메타 명령어·SQL 쿼리·설정 파일 세 가지 방법으로 조회하고, `ALTER SYSTEM SET` + `pg_reload_conf()`로 동적 변경한다.

## 2. 상세 설명

### 2-1. psql 메타 명령어 (가장 간편)

| 명령 | 설명 |
|------|------|
| `\show parameter_name` | 특정 설정값 즉시 출력 (`\show max_connections`) |
| `\show all` | 현재 적용된 모든 설정값 + 설명 목록 표시 |

### 2-2. SQL 쿼리 (SHOW / pg_settings)

```sql
-- 단순 확인 (psql 외 GUI 툴에서도 동작)
SHOW work_mem;
SHOW all;

-- 강력 추천: 단위·최솟값·최댓값·재시작 필요 여부까지 확인
SELECT name, setting, unit, context, short_desc
FROM pg_settings
WHERE name = 'max_connections';
```

> `context = 'postmaster'`인 항목은 수정 후 **서버 재시작**이 필요합니다.

### 2-3. 설정 파일 직접 확인

```sql
-- postgresql.conf 경로 확인
SHOW config_file;
-- 예: /var/lib/pgsql/data/postgresql.conf
```

파일을 `vi` 또는 `cat`으로 열면 주석 처리된 기본값과 현재 설정을 함께 확인할 수 있습니다.

---

### 2-4. 자주 확인하는 설정값

| 설정 이름 | 의미 |
|-----------|------|
| `port` | DB가 사용 중인 포트 번호 |
| `max_connections` | 최대 동시 접속 가능 유저 수 |
| `shared_buffers` | DB 엔진이 사용하는 공유 메모리 크기 |
| `data_directory` | 데이터가 실제로 저장되는 물리적 경로 |
| `log_directory` | 로그 파일이 저장되는 위치 |

---

### 2-5. 설정값 변경

```sql
-- 시스템 전체에 적용 (postgresql.auto.conf에 기록)
ALTER SYSTEM SET idle_in_transaction_session_timeout = '5min';

-- 서버 재시작 없이 즉시 반영 (context = 'sighup' 이하 설정만 가능)
SELECT pg_reload_conf();
```

- `ALTER SYSTEM SET`은 `postgresql.auto.conf`에 기록되며, `postgresql.conf`보다 우선 적용됩니다.
- `postmaster` context 설정은 `pg_reload_conf()`로 반영되지 않으므로 서버를 재시작해야 합니다.

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_pg_stat_activity-실행중인쿼리-확인]]
- 관련 링크: [[2026-06-02_PostgreSQL-통계정보-뷰-종류]]
- 관련 링크: [[2026-06-03_PostgreSQL-psql-주요명령어]]
