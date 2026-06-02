# MySQL 데이터이관 Shell Script

- **태그**: #MySQL #데이터이관 #mysqldump #ShellScript
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-mysql데이터이관shellscript.md`

## 1. 핵심 요약
- mysqldump 파이프라인을 Shell 함수로 캡슐화하여 테이블 목록 기반으로 스키마 생성·INSERT 이관·REPLACE 이관 세 가지 모드를 선택적으로 실행하는 MySQL 서버 간 데이터 이관 스크립트

## 2. 상세 설명

### 핵심 변수

| 변수 | 역할 |
|------|------|
| `S_HOST` / `T_HOST` | 원본·대상 서버 호스트명 또는 IP |
| `S_DATABASE` / `T_DATABASE` | 원본·대상 데이터베이스 이름 |
| `TBLLIST` | 이관 대상 테이블 목록 (공백 구분) |

### mysqldump 주요 옵션

| 옵션 | 설명 |
|------|------|
| `--add-drop-table` | 덤프 앞에 DROP TABLE 추가 — 대상 테이블 재생성 |
| `--no-create-info` | CREATE TABLE 제외 — 데이터만 이관 |
| `--skip-add-drop-table` | DROP TABLE 구문 제외 |
| `--replace` | INSERT 대신 REPLACE INTO 사용 — 중복 키 충돌 시 UPDATE |
| `--single-transaction` | InnoDB 일관성 스냅샷 덤프 (잠금 없음) |
| `--set-gtid-purged=OFF` | GTID 관련 SET 구문 제거 — 복제 환경 이관 오류 방지 |
| `--no-data` | 스키마(DDL)만 덤프, 데이터 제외 |
| `--where="조건"` | 조건에 맞는 행만 선택적 덤프 |

### 세 가지 이관 함수

```bash
# 1. 스키마만 이관 (데이터 없음)
crtonly_runsql

# 2. 스키마 + INSERT 이관 (중복 시 에러)
crt_runsql "WHERE 조건"   # 조건 생략 시 전체 이관

# 3. 스키마 + REPLACE 이관 (중복 시 UPDATE)
rep_runsql "WHERE 조건"   # 대상에 동일 PK 존재하면 덮어씀
```

### 파이프라인 흐름

```
mysqldump (source) → {TABLE}.dat → mysql (target)
```

각 테이블을 개별 `.dat` 파일로 덤프한 뒤 `mysql` 클라이언트로 즉시 임포트합니다.  
파이프 직연결(`|`) 대신 파일 경유 방식이므로 실패 시 `.dat` 파일로 재시도 가능합니다.

### 사용 예시

```bash
# TBLLIST에 T1, T2 설정 후 전체 이관
TBLLIST="T1 T2"
crt_runsql ""            # 전체 행 이관

# 특정 조건만 이관
crt_runsql "id > 1000"

# 이미 존재하는 데이터 갱신 포함
rep_runsql ""
```

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_Linux-고정IP-설정]]
