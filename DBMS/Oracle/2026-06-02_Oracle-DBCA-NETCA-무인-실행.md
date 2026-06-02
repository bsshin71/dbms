# Oracle DBCA·NETCA 무인 실행

- **태그**: #Oracle #DBCA #NETCA #설치 #DBA
- **작성일**: 2026-06-02
- **참조 원본**: `raw/clippings/2026-06-02-DBCA와NETCA 실행방법.md`

## 1. 핵심 요약
- Rocky Linux 9.x에서 Oracle 19c DBCA·NETCA를 응답 파일(`.rsp`) 기반으로 무인(silent) 실행하여 DB 인스턴스와 네트워크 리스너를 자동 구성하는 절차.

## 2. 상세 설명

### 사전 작업 — 데이터 디렉터리 선제 생성 (oracle 계정)

DBCA 무인 실행 시 디렉터리 부재로 검증 오류가 발생하므로, 데이터파일 저장 경로를 미리 수동 생성한다.

```bash
mkdir -p /oracle/u01/app/oracle/oradata
```

---

### Step 1. dbca.rsp 응답 파일 작성

Oracle 19c 내부 XML 스키마가 요구하는 **CamelCase·배치 순서** 규격을 정확히 맞춰야 한다.  
섹션 구분(`[...]`) 없이 홈 디렉터리에 생성한다.

```bash
cat << 'EOF' > /home/oracle/dbca.rsp
createDatabase=true
gdbName=devdb
sid=devdb
databaseConfigType=SI
templateName=General_Purpose.dbc
sysPassword=Oracle19c!
systemPassword=Oracle19c!
emConfiguration=NONE
dbsnmpPassword=Oracle19c!
characterSet=AL32UTF8
nationalCharacterSet=AL16UTF16
storageType=FS
datafileDestination=/oracle/u01/app/oracle/oradata
sampleSchema=false
automaticMemoryManagement=false
totalMemory=0
EOF
```

- SYS·SYSTEM 초기 비밀번호: `Oracle19c!`

---

### Step 2. DBCA 무인 DB 생성

```bash
source ~/.bash_profile
dbca -silent -createDatabase -responseFile /home/oracle/dbca.rsp
```

- 진행률 40~50% 구간에서 딕셔너리 뷰 수천 개를 컴파일하므로 장비 성능에 따라 10~20분 대기는 정상이다.
- `100% 완료` 및 `데이터베이스 생성이 완료되었습니다.` 메시지가 뜨면 성공.

---

### Step 3. NETCA 무인 리스너 생성

DBeaver·Toad 등 외부 툴 접속을 위한 1521 포트 리스너를 빌드한다.

```bash
netca -silent -responseFile $ORACLE_HOME/assistants/netca/netca.rsp
lsnrctl status
```

- `lsnrctl status` 결과에 `Instance "devdb", status READY`가 나오면 리스너가 DB 인스턴스를 정상 인식한 것이다.

---

### Step 4. DB 가동 상태 최종 검증

```sql
sqlplus / as sysdba

SELECT name, open_mode, database_role FROM v$database;
SELECT * FROM nls_database_parameters WHERE parameter='NLS_CHARACTERSET';

exit;
```

- 정상 조건: `OPEN_MODE = READ WRITE`, `VALUE = AL32UTF8`

---

### Step 5. 재부팅 시 오라클 자동 시작 설정

#### /etc/oratab 수정

```bash
vi /etc/oratab
# 끝자리 N → Y 로 변경
devdb:/oracle/u01/app/oracle/product/19.3.0/dbhome_1:Y
```

#### 수동 일괄 제어

```bash
dbshut $ORACLE_HOME   # DB + 리스너 일괄 종료
dbstart $ORACLE_HOME  # DB + 리스너 일괄 시작
```

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_RockyLinux9.7-Oracle19c-설치]]
- 관련 링크: [[2026-06-02_Oracle-Listener-등록]]
- 관련 링크: [[2026-06-02_Oracle-유저-생성]]
