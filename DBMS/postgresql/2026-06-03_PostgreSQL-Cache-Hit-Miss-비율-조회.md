# PostgreSQL Cache Hit / Miss 비율 조회

- **태그**: #PostgreSQL #모니터링 #통계정보 #캐시
- **작성일**: 2026-06-03
- **참조 원본**: `raw/clippings/2026-06-02-database 별 cache hit  miss 비율 조회.md`

## 1. 핵심 요약
- `pg_stat_database`의 `blks_hit`(버퍼 캐시에서 읽은 블록)과 `blks_read`(디스크에서 읽은 블록)를 조회하여 데이터베이스별 캐시 효율을 파악한다.

## 2. 상세 설명

### 조회 쿼리

```sql
SELECT datname, blks_hit, blks_read
FROM pg_stat_database;
```

### 컬럼 의미

| 컬럼 | 설명 |
|------|------|
| `datname` | 데이터베이스 이름 |
| `blks_hit` | 공유 버퍼 캐시에서 바로 읽은 블록 수 (디스크 I/O 없음) |
| `blks_read` | 실제 디스크(OS 캐시 포함)에서 읽은 블록 수 |

### 캐시 히트율 계산

```sql
SELECT datname,
       blks_hit,
       blks_read,
       ROUND(blks_hit::numeric / NULLIF(blks_hit + blks_read, 0) * 100, 2) AS hit_ratio_pct
FROM pg_stat_database
ORDER BY hit_ratio_pct;
```

- **히트율 = blks_hit / (blks_hit + blks_read) × 100**
- 일반적으로 **90% 이상**을 권장한다.
- 히트율이 낮으면 `shared_buffers` 증설 또는 쿼리 개선을 검토한다.

### 예시 결과 해석

| datname | blks_hit | blks_read | 비고 |
|---------|----------|-----------|------|
| postgres | 41246 | 420 | 히트율 ≈ 99% (양호) |
| pgbenchtest | 31885 | 14064 | 히트율 ≈ 69% (개선 필요) |
| hr | 29040 | 325 | 히트율 ≈ 99% (양호) |

`pgbenchtest`처럼 `blks_read`가 상대적으로 높으면 버퍼 미스가 잦다는 신호이며, 워크로드 패턴이나 `shared_buffers` 설정을 재검토해야 한다.

## 3. 연관 개념 (지식 연결)
- 관련 링크: [[2026-06-02_PostgreSQL-통계정보-뷰-종류]]
- 관련 링크: [[2026-06-03_PostgreSQL-시스템설정-Parameter-조회-변경]]
- 관련 링크: [[2026-06-03_PostgreSQL-DB용량-확인-쿼리]]
