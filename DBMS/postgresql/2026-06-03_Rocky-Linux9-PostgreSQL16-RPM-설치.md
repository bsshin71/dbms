# Rocky Linux 9 PostgreSQL 16 RPM 설치

- **태그**: #PostgreSQL #설치 #RockyLinux #RPM
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-Rocky9+PG설치 (rpm).md`

## 1. 핵심 요약
- CentOS 지원 종료 이후 Rocky Linux 9에서 PGDG RPM 패키지로 PostgreSQL 16을 설치하고, systemd 대신 커스텀 데이터 디렉토리로 직접 관리하는 전체 절차.

## 2. 상세 설명

### 설치 환경
- OS: Rocky Linux 9 (RHEL 호환)
- DB: PostgreSQL 16

### 설치 순서

**① PGDG 저장소 등록**
```bash
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

**② OS 내장 PostgreSQL 모듈 비활성화**
```bash
sudo dnf -qy module disable postgresql
```

**③ postgresql16-server 설치**
```bash
sudo dnf install -y postgresql16-server
```
설치 후 바이너리는 `/usr/pgsql-16/bin/`에 위치.

**④ 커스텀 디렉토리 구성 (systemd 미사용)**
```bash
mkdir -p /home/user/postgresql16/data
mkdir -p /home/user/postgresql16/logs

# bin 디렉토리 소프트링크
ln -s /usr/pgsql-16/bin /home/user/postgresql16/bin
```

**⑤ DB 초기화**
```bash
./postgresql16/bin/initdb -D /home/user/postgresql16/data
```
초기화 시 로케일 `en_US.UTF-8`, 타임존 `Asia/Seoul`, `shared_buffers=128MB`로 설정.

**⑥ postgresql.conf 주요 설정**
```ini
log_directory = '/home/user/postgresql16/logs'
log_filename = '%Y-%m-%d.log'
unix_socket_directories = '/home/user/postgresql16/data'
listen_addresses = '*'   # 원격 클라이언트 접속 허용
```

**⑦ 서버 시작**
```bash
./postgresql16/bin/pg_ctl -D /home/user/postgresql16/data start
```

**⑧ 환경변수 설정** (`.bash_profile`)
```bash
export PGHOST=/home/user/postgresql16/data
```
`PGHOST`에 소켓 디렉토리를 지정해야 `psql` 기본 접속이 가능.

**⑨ 사용자 DB 생성 및 접속 확인**
```bash
./postgresql16/bin/createdb user
psql
```

**⑩ 방화벽 포트 오픈 (필요 시)**
```bash
sudo firewall-cmd --permanent --zone=public --add-port=5432/tcp
sudo firewall-cmd --reload
```

### 보안 참고
- 초기화 직후 로컬 접속은 `trust` 인증 — `pg_hba.conf`에서 `scram-sha-256`으로 변경 권장.

## 3. 연관 개념 (지식 연결)
- [[2026-06-01_MinTool4PG-설치법]]
- [[2026-06-02_pgvector-설치]]
- [[2026-06-03_PostgreSQL18-RHEL8-RPM-설치]]
