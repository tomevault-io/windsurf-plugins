---
trigger: always_on
description: > SQLite를 저장 엔진으로 쓰는 RESP 호환 서버.
---

# CLAUDE.md — bun-resp-sqlite

> SQLite를 저장 엔진으로 쓰는 RESP 호환 서버.
> 클라이언트는 항상 순정 `Bun.RedisClient`로 접속한다.
> 이 문서는 **설계의 단일 진실(SSOT)** 이다. 구현은 Claude Code에서 별도로 진행한다.

---

## 0. 한 줄 정의

`Bun.RedisClient`(RESP3 클라이언트)가 보내는 wire protocol을, Bun TCP 서버가 받아 SQLite로 처리하고 RESP로 응답한다. **서버는 클라이언트를 흉내 내지 않는다 — 프로토콜에 응답할 뿐이다.**

```
┌────────────────────┐   RESP3 over TCP    ┌──────────────────────────────┐
│  Bun.RedisClient   │ ──────────────────▶ │  bun-resp-sqlite (this)      │
│  (순정, 무수정)     │ ◀────────────────── │  Bun.listen() + bun:sqlite   │
└────────────────────┘   RESP3 replies      └──────────────────────────────┘
       애플리케이션                                   같은 머신/별도 프로세스
```

---

## 1. 목적과 비목적 (Scope SSOT)

### 목적
- `Bun.RedisClient`가 **코드 수정 없이** 접속·동작한다. 접속 URL만 이 서버를 가리키면 끝.
- Redis 서버 **설치 없이** Redis를 쓴다. 의존성은 Bun 런타임 하나(`bun:sqlite`, `Bun.listen` 모두 내장).
- 데이터는 SQLite 파일에 영속화된다(프로세스 재시작 후 생존).

### 비목적 (명시적으로 하지 않는 것)
- 인프로세스 라이브러리 모드(클라이언트 시그니처 직접 구현)는 **만들지 않는다.** 접속은 무조건 RESP over TCP.
- Redis Cluster / Sentinel은 지원하지 않는다 (`Bun.RedisClient` 자체가 미지원).
- 다중 노드 HA·자동 failover는 범위 밖.
- 인메모리 전용 Redis의 모든 성능 특성을 동일 재현하지 않는다 — **인터페이스 호환이 목표지 성능 동등이 아니다.**

---

## 2. 호환성 계약 (Compatibility Contract — 가장 중요한 SSOT)

호환성의 기준은 **"클라이언트 메서드를 다 구현했는가"가 아니라 "클라이언트가 보내는 바이트에 올바른 바이트로 답하는가"** 이다. 아래 4개 계층을 모두 만족해야 "완전 호환"이 성립한다.

### 2.1 프로토콜 계층 — RESP3 필수
`Bun.RedisClient`는 RESP3로 말한다(Zig 구현). 따라서:
- 서버는 `HELLO 3` 핸드셰이크에 응답해야 한다. RESP2만 구현하면 응답 타입이 어긋난다.
- RESP3 고유 타입을 낼 수 있어야 한다: Null(`_`), Boolean(`#`), Map(`%`), Set(`~`), Double(`,`), Verbatim 등.
- 클라이언트의 자동 타입 변환 규칙(아래 §2.4)을 서버 응답이 유발해야 한다.

### 2.2 핸드셰이크/연결 계층 — 연결 성립의 전제
클라이언트가 connect 시 호출하는 명령은 **명령 처리 이전에** 반드시 응답해야 한다. 하나라도 빠지면 핸드셰이크에서 막힌다.

| 명령 | 역할 | 1차 구현 |
|---|---|---|
| `HELLO [3] [AUTH ...]` | 프로토콜 협상 + 서버 정보 Map 반환 | **필수** |
| `AUTH` | 인증 (URL에 자격증명 있을 때) | **필수** (no-auth면 OK 반환) |
| `PING` | 헬스체크 / keepalive | **필수** |
| `SELECT` | DB 번호 선택 (URL `/0`) | **필수** (단일 DB여도 OK 반환) |
| `INFO` | 서버 메타데이터 | **필수** (최소 필드) |
| `QUIT` | 연결 종료 | **필수** |
| `CLIENT` | 클라이언트 설정 (`CLIENT SETINFO` 등) | 권장 (OK 반환) |

> 이 명령들은 클라이언트 문서상 **자동 파이프라이닝이 비활성화**되는 명령군에 속한다(`AUTH`/`INFO`/`QUIT`/`SELECT`/`MULTI`/`EXEC`/`WATCH` 등). 서버는 이들을 단건 동기 응답으로 처리한다고 가정해도 된다.

### 2.3 명령 계층 — 커버리지가 곧 호환성
전용 메서드가 있는 명령은 반드시 지원한다. 나머지는 클라이언트가 `send(CMD, args[])`로 raw 전송하므로, **서버 입장에서는 전용/raw 구분이 없다 — 모두 같은 RESP 배열로 도착한다.** 즉 호환성 확장 = 명령 디스패치 테이블에 케이스 추가.

전용 메서드 보유 명령(문서 확인됨, **1차 핵심 대상**):
- String: `GET` `SET` `GETSET`(via send) `DEL` `EXISTS` `GETBUFFER`(=GET, 바이너리 응답)
- Numeric: `INCR` `DECR`
- Expire: `EXPIRE` `TTL`
- Hash: `HSET` `HMSET` `HGET` `HMGET` `HINCRBY` `HINCRBYFLOAT`
- Set: `SADD` `SREM` `SISMEMBER` `SMEMBERS` `SRANDMEMBER` `SPOP`
- Multi-key: `MGET` `MSET` `MSETNX` `SETEX` `SETNX`

### 2.4 응답 타입 변환 계약 — 어긋나면 "호환"이 깨지는 지점
클라이언트는 RESP 응답을 JS 값으로 자동 변환한다. 서버는 **정확히 그 변환을 유발하는 RESP 타입**을 내야 한다. 이게 호환성의 가장 미묘한 부분.

| 클라이언트 기대 JS | 서버가 보낼 RESP 타입 |
|---|---|
| number | Integer (`:`) |
| string | Bulk/Simple String (`$`/`+`) |
| `null` | Null Bulk / RESP3 Null (`_`) |
| array | Array (`*`) |
| boolean | RESP3 Boolean (`#`) |
| object(map) | RESP3 Map (`%`) |
| array(set) | RESP3 Set (`~`) |
| Error throw | Error (`-`) + 코드 |

**명령별 특수 변환(반드시 준수):**
- `EXISTS` → 정수 1/0이 아니라 **boolean**으로 변환됨. 서버는 RESP3 Boolean으로 답해야 클라이언트 기대와 일치. (또는 클라이언트가 정수→boolean 변환을 보장하는지 통합 테스트로 고정)
- `SISMEMBER` → 동일하게 boolean.
- `getBuffer()` → 같은 `GET`이지만 Uint8Array로 받음. 서버는 동일 Bulk String을 내되 **바이너리 안전**해야 한다(임의 바이트 보존).

**에러 코드 계약:** 클라이언트는 `error.code`로 분기한다. 서버 에러 응답은 클라이언트가 아는 코드로 매핑되어야 한다:
- `ERR_REDIS_CONNECTION_CLOSED`, `ERR_REDIS_AUTHENTICATION_FAILED`, `ERR_REDIS_INVALID_RESPONSE`.
- 일반 명령 에러는 표준 RESP 에러 프리픽스(`ERR`, `WRONGTYPE`, `WRONGPASS` 등)로.

---

## 3. 아키텍처 (메타 설계)

### 3.1 레이어 경계
각 레이어는 한 가지 책임만 진다. 위→아래 단방향 의존.

```
┌─────────────────────────────────────────────────────────────┐
│ L1  Transport      Bun.listen() TCP 서버 / 소켓 수명주기      │
│        ▼            (바이트 in/out, 연결당 상태)               │
│ L2  RESP Codec     RESP3 파서(스트리밍) + 직렬화기            │
│        ▼            (바이트 ↔ Command / Reply)                │
│ L3  Connection     연결당 상태기계: handshake→ready→subscribe │
│        ▼            (HELLO/AUTH/SELECT, 모드 전환)             │
│ L4  Dispatcher     명령 라우팅 테이블 (CMD → Handler)         │
│        ▼            (arity 검증, 미지원 명령 에러)            │
│ L5  Command Engine 명령별 의미론 (KV/Hash/Set/Expire...)      │
│        ▼                                                       │
│ L6  Storage        StorageEngine 인터페이스 (추상)            │
│        ▼            └─ SqliteStorage (bun:sqlite, WAL)        │
│ L7  Side-systems   ExpiryReaper / PubSubHub / TxnContext     │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 의존성 역전 — Storage는 추상 경계
`Command Engine`은 SQLite를 모른다. `StorageEngine` 인터페이스에만 의존한다. 이렇게 두면 (1) 테스트 시 인메모리 mock 교체, (2) 후일 다른 저장 엔진 실험이 열린다. **단, 이 추상화에 Redis 고유 개념을 그대로 노출하지 않는다** — KV/필드맵/정렬셋 같은 저장 원형(primitive)만 올린다.

```
StorageEngine (interface = 저장 SSOT)
  ├─ kvGet/kvSet/kvDel/kvExists
  ├─ fieldGet/fieldSet/fieldDel        (hash 계열의 저장 원형)
  ├─ memberAdd/memberRem/memberHas     (set 계열의 저장 원형)
  ├─ expireSet/expireGet/sweepExpired  (TTL)
  └─ withTransaction(fn)               (원자 단위)
        │
        └─ SqliteStorage  (bun:sqlite, WAL 모드)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Munsunty/bundis](https://github.com/Munsunty/bundis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
