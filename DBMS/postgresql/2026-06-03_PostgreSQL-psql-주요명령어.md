# PostgreSQL psql 주요 명령어

- **태그**: #PostgreSQL #psql #메타명령어 #DBA도구
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-psql  주요 명령어.md`

## 1. 핵심 요약
- psql 클라이언트에서 `\`로 시작하는 메타 명령어로 DB·스키마·테이블·뷰·유저 정보를 SQL 없이 빠르게 조회할 수 있다.

## 2. 상세 설명

### 데이터베이스 목록

| 명령 | 설명 |
|------|------|
| `\l` | 모든 데이터베이스 목록(이름·소유주·인코딩·로케일·권한) 출력 |

---

### 스키마 목록

| 명령 | 설명 |
|------|------|
| `\dn *` | 시스템 스키마를 포함한 모든 스키마 출력 |
| `\dnS` | 시스템 스키마만 출력 |
| `\dS` | 시스템 카탈로그 테이블 목록 출력 |

---

### 테이블 목록

| 명령 | 설명 |
|------|------|
| `\dt` | 현재 `search_path` 내 사용자 테이블 목록 |
| `\dt *.*` | 모든 스키마의 테이블 목록 |
| `\dt pg_catalog.*` | 시스템 테이블 목록만 |
| `\dt+` | 테이블 목록 + 크기(Size) 및 설명 추가 |
| `\dt user_*` | 이름이 `user_`로 시작하는 테이블만 |
| `\d my_table` | 특정 테이블의 컬럼·타입·인덱스 등 구조 확인 |

SQL 대안:
```sql
SELECT schemaname, tablename, tableowner
FROM pg_catalog.pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema');
```

---

### 뷰(View) 목록

| 명령 | 설명 |
|------|------|
| `\dv` | 현재 `search_path` 내 사용자 뷰 목록 |
| `\dv *.*` | 시스템 뷰 포함 모든 뷰 |
| `\dv pg_catalog.*` | 시스템 관리 뷰만 (`pg_stat_activity` 등) |
| `\dv+` | 뷰 목록 + 설명 추가 |
| `\d+ 뷰_이름` | 뷰 컬럼 정보 + 원본 SELECT 문 확인 |

SQL 대안:
```sql
-- 사용자가 만든 일반 뷰
SELECT schemaname, viewname, viewowner
FROM pg_views
WHERE schemaname NOT IN ('pg_catalog', 'information_schema');

-- pg_stat으로 시작하는 시스템 상태 뷰
SELECT viewname
FROM pg_views
WHERE schemaname = 'pg_catalog' AND viewname LIKE 'pg_stat%';
```

---

### 유저(Role) 목록

| 명령 | 설명 |
|------|------|
| `\du` | 모든 사용자의 이름·속성·소속 그룹 출력 |
| `\du+` | 사용자 목록 + 설명(Description) 추가 |

SQL 대안:
```sql
-- 기본 사용자 정보
SELECT usename AS user_name, usesuper AS is_superuser, usecreatedb AS can_create_db
FROM pg_user;

-- 역할(Role) 상세 정보
SELECT rolname, rolsuper, rolinherit, rolcreaterole, rolcanlogin
FROM pg_authid;
```

현재 접속 계정 확인:
```sql
SELECT current_user;  -- 현재 세션의 유저
SELECT session_user;  -- 로그인한 실제 유저
```

---

### 목적별 명령어 요약

| 목적 | 명령어 |
|------|--------|
| 테이블 조회 | `\dt` |
| 뷰 조회 | `\dv` |
| 테이블·뷰·인덱스·시퀀스 모두 | `\d` |
| 시스템 객체까지 포함 | 뒤에 `*.*` 추가 |
| 상세 정보 추가 | 뒤에 `+` 추가 |

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-03_PostgreSQL-시스템설정-Parameter-조회-변경]]
- 관련 링크: [[2026-06-03_PostgreSQL-인덱스-상세정보-조회-di+]]
- 관련 링크: [[2026-06-02_PostgreSQL-통계정보-뷰-종류]]
- 관련 링크: [[2026-06-02_pg_stat_activity-실행중인쿼리-확인]]
