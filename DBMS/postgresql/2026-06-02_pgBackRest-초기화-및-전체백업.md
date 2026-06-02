# pgBackRest 초기화 및 전체 백업 수행 절차

- **태그**: #PostgreSQL #백업 #pgBackRest #운영
- **작성일**: 2026-06-02
- **참조 원본**: `raw/clippings/2026-06-02-pgBackRest 초기화 및 전체 백업 수행 절차.md`

## 1. 핵심 요약
- pgBackRest에서 Stanza 생성 → 설정 검증 → 전체 백업 실행 → 결과 확인의 4단계로 PostgreSQL 백업 시스템을 구축한다.

## 2. 상세 설명

### 사전 준비 (Prerequisites)
`postgresql.conf`에 pgBackRest 아카이빙 설정이 완료되어 있어야 하며, 설정 변경 후에는 PostgreSQL 재시작이 필요합니다.

### 단계 1: Stanza 생성 (Create Stanza)
Stanza는 특정 데이터베이스 클러스터에 대한 백업 설정 단위입니다. `pgbackrest.conf` 설정을 바탕으로 백업 저장소 구조를 초기화합니다.

```bash
# postgres 계정으로 실행
pgbackrest --stanza=mydb stanza-create
```

### 단계 2: 설정 검증 (Check)
PostgreSQL과 pgBackRest 간 통신 및 아카이브 경로 정상 여부를 확인합니다.

```bash
pgbackrest --stanza=mydb check
```

성공 시 `status: ok` 메시지가 출력됩니다. 오류 발생 시 `postgresql.conf`의 `archive_command` 설정을 재확인합니다.

### 단계 3: 전체 백업 실행 (Full Backup)
최초 백업은 반드시 `--type=full`을 지정합니다.

```bash
pgbackrest --stanza=mydb backup --type=full
```

### 단계 4: 백업 결과 확인 (Info)

```bash
pgbackrest info
```

### 관리 및 자동화

**증분 백업**: 전체 백업 이후에는 변경분만 저장합니다.

```bash
pgbackrest --stanza=mydb backup --type=incr
```

**crontab 자동화 예시**: 매주 일요일 전체 백업, 매일 자정 증분 백업

**보관 정책**: `repo1-retention-full=2` 설정 시 전체 백업 최신 2개 유지, 나머지 자동 expire

### ⚠️ 테이블스페이스 경로 주의사항
`pg_tblspc` 내의 심볼릭 링크가 `PGDATA` 외부를 가리켜야 `[070] ERROR`를 방지할 수 있습니다.

```bash
# 확인용 명령어
ls -al /home/postgres/pg18/data/pg_tblspc
# 결과: /home/postgres/pg18/ts_data/... 처럼 data 폴더 외부를 향해야 함
```

복구(Restore) 테스트까지 함께 진행하는 것을 권장합니다.

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-01_MinTool4PG-설치법]]
- 관련 링크: [[2026-06-02_pgvector-설치]]
