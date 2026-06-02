# Oracle 리스너 동적·정적 등록

- **태그**: #Oracle #Listener #네트워크설정 #DBA
- **작성일**: 2026-06-02
- **참조 원본**: `raw/clippings/2026-06-02-listener 등록.md`

## 1. 핵심 요약
- 오라클 DB는 기동 시 LREG 백그라운드 프로세스가 자동으로 리스너에 등록(동적 등록)되므로 `listener.ora` 수정이 불필요하며, **DB가 꺼진 상태에서 원격 기동**이 필요한 경우에만 정적 등록을 추가한다.

## 2. 상세 설명

### 동적 등록 (Dynamic Service Registration) — 기본 동작

오라클 DB 인스턴스가 `STARTUP`되면 **LREG(Listener Registration)** 프로세스가 다음 순서로 동작한다.

1. 기본 포트 **1521**로 리스너가 살아있는지 탐색
2. 리스너 감지 시, 자신의 서비스명·SID·상태를 리스너 메모리에 주입
3. 기동 후 **30초~1분 이내** 자동 완료 → `listener.ora` 무수정으로 외부 접속 가능

> `lsnrctl status`에서 해당 서비스가 `READY` 상태로 나타나는 것이 정상이다.

### 정적 등록 (Static Registration) — 예외 상황

| 상황 | 동적 등록 | 정적 등록 |
|------|-----------|-----------|
| DB 기동 중 | 가능 | 가능 |
| DB 종료 후 원격 접속 | **불가** | **가능** |

정적 등록이 필요한 유일한 실무 상황: **"DB가 SHUTDOWN된 상태에서 원격 클라이언트(sqlplus 등)로 접속해 DB를 켜야 할 때"**

DB가 종료되면 동적 등록 정보가 리스너에서 사라지므로, 원격에서 `sqlplus sys/pw@IP:1521/SID as sysdba`로 접속하려 해도 리스너가 해당 SID를 알지 못한다.

### listener.ora 정적 등록 예시

```ini
# /oracle/u01/app/oracle/product/19.3.0/dbhome_1/network/admin/listener.ora

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = Rocky9.7)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )

# 정적 등록: DB가 꺼진 상태에서도 리스너에 고정
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = devdb)
      (ORACLE_HOME = /oracle/u01/app/oracle/product/19.3.0/dbhome_1)
      (SID_NAME = devdb)
    )
    (SID_DESC =
      (GLOBAL_DBNAME = proddb)
      (ORACLE_HOME = /oracle/u01/app/oracle/product/19.3.0/dbhome_1)
      (SID_NAME = proddb)
    )
  )
```

수정 후 `lsnrctl reload`로 반영하면 `lsnrctl status`에서 상태가 `UNKNOWN`으로 표시된다. (`UNKNOWN` = 정적 등록의 정상 상태)

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-01_MinTool4PG-설치법]]
