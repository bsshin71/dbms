# PostgreSQL 18 RHEL 8 RPM 설치

- **태그**: #PostgreSQL #설치 #RHEL8 #RPM
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-postresql설치(ver 18).md`

## 1. 핵심 요약
- RHEL 8 계열에서 PGDG 공식 저장소로 PostgreSQL 18을 RPM 설치하고, 데이터 디렉토리를 커스텀 경로로 이전하며 SELinux·외부 접속까지 구성하는 전체 절차.

## 2. 상세 설명

### 설치 환경
- OS: RHEL 8 계열 (Rocky Linux 8, AlmaLinux 8 등)
- DB: PostgreSQL 18

### 설치 순서

**① 기존 저장소 파일 제거**
```bash
rm -f /etc/yum.repos.d/pgdg*.repo
dnf clean all
```
이전에 잘못 등록된 저장소가 남아 있으면 XML URL 오류가 발생하므로 먼저 제거.

**② PGDG 공식 저장소 RPM 설치 (EL-8)**
```bash
dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```

**③ OS 내장 PostgreSQL 모듈 비활성화**
```bash
dnf -qy module disable postgresql
```
RHEL 8 AppStream의 기본 postgresql 모듈과 PGDG 패키지가 충돌하는 것을 방지.

**④ postgresql18 설치**
```bash
dnf install -y postgresql18-server postgresql18
```
바이너리는 `/usr/pgsql-18/bin/`에 위치.

**⑤ DB 초기화 및 서비스 활성화**
```bash
/usr/pgsql-18/bin/postgresql-18-setup initdb
systemctl enable --now postgresql-18
```
기본 데이터 디렉토리: `/var/lib/pgsql/18/data/`

---

### 데이터 디렉토리 이전 (커스텀 경로)

**① 서비스 중지**
```bash
systemctl stop postgresql-18
```

**② 새 디렉토리 생성 및 데이터 복사**
```bash
mkdir -p /home/postgres/pg18/data
cp -a /var/lib/pgsql/18/data/* /home/postgres/pg18/data/
```

**③ 권한 설정**
```bash
chown -R postgres:postgres /home/postgres/pg18
chmod 700 /home/postgres/pg18/data          # 데이터 디렉토리: postgres만 접근 가능
chmod 600 /home/postgres/pg18/data/*.conf   # 설정 파일
```

**④ systemd PGDATA 변경**

`systemctl edit postgresql-18`으로 편집기를 열어 아래 내용 추가:
```ini
[Service]
Environment=PGDATA=/home/postgres/pg18/data
```

**⑤ SELinux 보안 컨텍스트 설정 (RHEL 8 필수)**

`/home` 경로에 PostgreSQL 프로세스가 접근하려면 SELinux 라벨이 필요.
```bash
dnf install -y policycoreutils-python-utils

semanage fcontext -a -t postgresql_db_t "/home/postgres/pg18/data(/.*)?"
restorecon -R -v /home/postgres/pg18/data
```

**⑥ 설정 반영 및 서비스 재시작**
```bash
systemctl daemon-reload
systemctl start postgresql-18
```

---

### 외부 접속 허용

**postgresql.conf**
```ini
listen_addresses = '*'
```

**pg_hba.conf**
```
host all all 0.0.0.0/0 scram-sha-256
```

**서비스 재시작**
```bash
systemctl restart postgresql-18
```

### postgres 계정 비밀번호 설정
```bash
su - postgres -c "psql -c \"ALTER USER postgres PASSWORD '사용할비밀번호';\""
```

### 트러블슈팅 — 포어그라운드 기동
서비스 시작 오류 시 포어그라운드로 실행하여 오류 메시지 직접 확인:
```bash
su - postgres -c "/usr/pgsql-18/bin/postgres -D /home/postgres/pg18/data"
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-03_Rocky-Linux9-PostgreSQL16-RPM-설치]]
- [[2026-06-01_MinTool4PG-설치법]]
- [[2026-06-02_pgvector-설치]]
