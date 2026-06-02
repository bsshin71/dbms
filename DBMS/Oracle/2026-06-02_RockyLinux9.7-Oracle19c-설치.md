# Rocky Linux 9.7 환경의 Oracle 19c 설치

- **태그**: #Oracle #설치 #RockyLinux #DBA
- **작성일**: 2026-06-02
- **참조 원본**: `raw/clippings/2026-06-02-RockyLinux9.7+Oracle 19.3.0.0설치.md`

## 1. 핵심 요약
- Rocky Linux 9.x에서 Oracle 19.3.0.0 GUI 설치는 폐기된 라이브러리 심볼릭 링크 생성, GCC 11 호환 컴파일 플래그 주입, 링크 에러 발생 시 수동 relink의 3단계 핵심 패치로 완료한다.

## 2. 상세 설명

### [Prerequisite] 필수 OS 패키지 및 호환 패치 (root 계정)

Oracle 기동에 필요한 패키지와, Rocky 9에서 폐기된 구형 라이브러리의 가짜 심볼릭 링크를 미리 생성한다.

```bash
# 1. 오라클 필수 패키지 및 xterm GUI 라이브러리 설치
dnf install -y BC binutils compat-openssl11 elfutils-libelf elfutils-libelf-devel \
  fontconfig-devel glibc glibc-devel ksh libaio libaio-devel \
  libXrender libXrender-devel libX11 libX11-devel libXext libXext-devel \
  libXtst libXtst-devel libgcc libstdc++ libstdc++-devel \
  libxcrypt-compat make sysstat targetcli smartmontools

dnf install -y xdpyinfo xorg-x11-apps

# 2. ★ 핵심 패치: Rocky 9에서 폐기된 libpthread_nonshared.a 심볼릭 링크 생성
rm -f /usr/lib64/libpthread_nonshared.a
ln -s /usr/lib64/libpthread.so.0 /usr/lib64/libpthread_nonshared.a
ldconfig

# 3. GUI 화면 소유권 전역 허용 (데스크톱 터미널에서 수행)
xhost +
```

### [Step 1] 환경 변수 설정 및 압축 해제 (oracle 계정)

```bash
# ~/.bash_profile 핵심 변수
export ORACLE_BASE=/oracle/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/19.3.0/dbhome_1
export ORA_INVENTORY=/oracle/u01/app/oraInventory
export ORACLE_SID=devdb
export TNS_ADMIN=$ORACLE_HOME/network/admin
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
export PATH=$ORACLE_HOME/bin:$ORACLE_HOME/OPatch:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
export CV_ASSUME_DISTID=RHEL9   # Rocky를 RHEL로 인식시키는 핵심 변수
export LANG=C
export DISPLAY=:0.0

source ~/.bash_profile

# Oracle Home 디렉터리 생성 후 설치 파일 압축 해제
mkdir -p $ORACLE_HOME
cd $ORACLE_HOME
unzip -q /home/oracle/install/V982063-01.zip
```

> `CV_ASSUME_DISTID=RHEL9`는 설치 프로그램이 Rocky Linux를 RHEL로 인식하게 강제하는 필수 변수다.

### [Step 2] 설치 전 Makefile 패치 (oracle 계정)

GCC 11의 엄격한 링커 검사를 우회하기 위해 `ins_rdbms.mk`에 하위 호환성 파라미터를 주입한다.

```bash
cd $ORACLE_HOME/rdbms/lib
sed -i 's/\$(DL_OPTS)/\$(DL_OPTS) -Wl,--no-as-needed/g' ins_rdbms.mk
```

### [Step 3] GUI 설치 마법사 실행

```bash
cd $ORACLE_HOME
./runInstaller
```

마법사 선택 가이드:

| 단계 | 선택 값 |
|------|---------|
| Configure Option | `Set up Software Only` |
| Database Installation Options | `Single instance database installation` |
| Database Edition | `Enterprise Edition` |
| Installation Locations | 환경 변수 경로와 일치 확인 |
| Prerequisite Checks | 구형 패키지 경고 → **우측 상단 `[ ] Ignore All` 체크 후 Next** |

### [Step 4] 링크 에러 발생 시 수동 패치 (핵심 승부처)

`Link binaries` 단계(약 11%~70%)에서 다음 에러가 발생하며 설치가 대기 상태에 걸린다.

> *Error in invoking target 'libasmclntsh19.ohso libasmperl19.ohso client_sharedlib' of makefile...*

**경고창을 그대로 두고(Abort/Retry/Continue 클릭 금지)** 새 터미널을 열어 수동 패치를 수행한다.

```bash
# 1. 방해꾼 구형 stub 파일 임시 제거
cd /oracle/u01/app/oracle/product/19.3.0/dbhome_1/lib/stubs
mv libc.so libc.so.bak

# 2. 핵심 클라이언트 공유 라이브러리 수동 선행 컴파일
cd /oracle/u01/app/oracle/product/19.3.0/dbhome_1/network/lib
export LDFLAGS="-Wl,--copy-dt-needed-entries -Wl,--no-as-needed"
make -f ins_net_client.mk client_sharedlib

# 3. GCC 11 완화 플래그를 적용한 전체 relink
cd /oracle/u01/app/oracle/product/19.3.0/dbhome_1/bin
export CFLAGS="-O2 -fpermissive -Wno-error=implicit-function-declaration"
export CXXFLAGS="-O2 -fpermissive -Wno-error=implicit-function-declaration"
export LDFLAGS="-Wl,--copy-dt-needed-entries -Wl,--no-as-needed"
./relink all
```

`./relink all`이 에러 없이 완료되면 GUI 에러 팝업의 `[Retry]`를 클릭한다. 게이지가 100%까지 진행되며 설치가 완료된다.

### [Step 5] 포스트 권한 스크립트 실행 (root 계정)

```bash
/oracle/u01/app/oraInventory/orainstRoot.sh
/oracle/u01/app/oracle/product/19.3.0/dbhome_1/root.sh
```

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_Oracle-DBCA-NETCA-무인-실행]]
- 관련 링크: [[2026-06-02_Oracle-Listener-등록]]
- 관련 링크: [[2026-06-02_Oracle-유저-생성]]
