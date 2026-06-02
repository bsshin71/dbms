# PostgreSQL DB 용량 확인 쿼리

- **태그**: #PostgreSQL #시스템카탈로그 #용량확인 #관리쿼리
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-PostgreSQL 데이터베이스 용량 확인을 위한 SQL 쿼리.md`

## 1. 핵심 요약
- `pg_database_size()`와 `pg_size_pretty()`를 조합하여 서버 내 모든 데이터베이스의 실제 디스크 사용량을 사람이 읽기 쉬운 단위(KB/MB/GB)로 조회하는 쿼리.

## 2. 상세 설명

### 기본 조회 쿼리

```sql
SELECT 
    oid, 
    pg_database.datname AS database_name, 
    pg_size_pretty(pg_database_size(pg_database.datname)) AS size 
FROM pg_database;
```

### 컬럼 설명

| 컬럼/함수 | 설명 |
|-----------|------|
| `oid` | 데이터베이스의 고유 식별자(Object Identifier) |
| `datname` | 데이터베이스 이름 (`database_name` 별칭 사용) |
| `pg_database_size(datname)` | 특정 DB의 실제 디스크 사용량을 **바이트(Byte)** 단위로 반환 |
| `pg_size_pretty(bytes)` | 바이트 값을 **KB / MB / GB** 등 가독성 좋은 단위로 변환 |
| `pg_database` | 서버 내 모든 데이터베이스 정보를 담는 시스템 카탈로그 테이블 |

### 활용 팁
- 특정 DB만 조회하려면 `WHERE datname = 'mydb'`를 추가합니다.
- 용량 순 정렬이 필요하면 `ORDER BY pg_database_size(datname) DESC`를 사용합니다.
- `pg_size_pretty()` 없이 바이트 값 그대로 비교·정렬하는 게 더 정확합니다.

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_PostgreSQL-통계정보-뷰-종류]]
- 관련 링크: [[2026-06-02_pg_stat_activity-실행중인쿼리-확인]]
- 관련 링크: [[2026-06-03_PostgreSQL-Tablespace-사용-객체-조회]]
