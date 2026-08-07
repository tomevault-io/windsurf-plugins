---
trigger: always_on
description: Unified OpenAI-compatible API gateway for **Grok + CodeBuddy**. Routes by model prefix:
---

# FoxRouters

## Project Overview
Unified OpenAI-compatible API gateway for **Grok + CodeBuddy**. Routes by model prefix:
`grok-*` → cli-chat-proxy.grok.com, `cb/*` → www.codebuddy.ai/v2.

Multi-account/key round-robin, auto-refresh (singleflight + pre-warm), circuit breaker,
API key auth, per-key RPM/quota, Redis hot-state, **ClickHouse** full-body history, web dashboard.

**Version:** v1.6.2 (`-X main.Version` build flag)
**Port:** 20130 · **Deploy:** Docker Compose (`docker compose up -d --build foxrouters`)  
**Path:** `/root/nexus-workspace/foxrouters/`

## Architecture / Flow

```
Client → AuthMiddleware (Bearer) → RateLimitMiddleware
       → /v1/chat/completions
            ↓
       proxyRequest (model routing + expandGrokAlias)
       ├── grok-* → proxyGrok (O(k) RR, 401 retry, 403 ban/cooldown + Redis persist)
       └── cb/*   → proxyCodeBuddy (stream-only transform, credit/14018 disable + Redis)
            ↓
       async LogRequest → ClickHouse (full body, ZSTD, unlimited)
```

### Data stores

| Layer | Engine | Purpose |
|-------|--------|---------|
| Hot | **Redis** | Tokens, CB credits, disabled flags, gateway keys, rate state, **proxy pool** |
| Cold | **LogStore** (pluggable via `LOG_BACKEND`) | `request_logs` full request/response JSON, refresh/events, 90d TTL |
| Legacy | PostgreSQL | **Not used** by gateway for history (may remain on disk) |

Log backend choices (`LOG_BACKEND` env, default `sqlite`):

| Backend | When to use | Footprint |
|---------|-------------|-----------|
| `sqlite` (default) | Small deployments; no ops overhead | Single file at `LOG_SQLITE_PATH` (default `/var/lib/foxrouters/logs.db`), ~60MB total |
| `clickhouse`       | Analytics workloads, high-volume queries | Separate CH server, ~700MB image + RAM |

### Hot-path rules (do not regress)
1. `Next()` = O(k) RR only — re-enable in background workers only  
2. Counts via `Len()`, not `len(GetAll())`  
3. Refresh = singleflight + lock-split (no network under `acc.mu`)  
4. Any disable/enable/token mutate → `Save*()` after unlock  
5. History write async only; credentials never in CH  
6. Full body unlimited in CH; log `id` JSON **string** for browsers  
7. No live gateway key inject into `/dashboard` HTML  
8. Proxy pool: `getClient(default, upstream)` — returns proxied client if pool has enabled proxies matching upstream scope, else direct. Transport cache per proxy ID. Auto-disable after 5 fails.

### Token refresh
- **Grok:** Pre-warm every 30s, 30min window, 10 concurrent  
- `Next()` Pass1 valid token; Pass2 least-expired + async refresh  
- 401 rebuild request body + retry  
- **CB OAuth:** Pre-warm worker (30s tick, 30m window) + `EnsureValid` before chat + 401 refresh-retry. Singleflight + lock-split. Refresh via `POST /v2/plugin/auth/token/refresh` (`X-Refresh-Token`). Eager refresh on import when AT is expired/near-expiry and RT is valid.

### CB dual pool (`api_key` + `oauth`)
- Same chat endpoint: `www.codebuddy.ai/v2/chat/completions`
- Mixed round-robin over one `CBKey` pool (`cred_type`: `api_key` | `oauth`)
- Upstream auth: API key = Bearer or `X-API-Key`; OAuth = `Authorization: Bearer <AT>` only
- **Credit sync:** worker every 5m + `POST /cb/credits/sync`. Meter API `POST /v2/billing/meter/get-user-resource` (works for both modes). Persist limit/remain/package/cycle/status. Permanent disable on `Status==3`. Fallback `CB_CREDIT_LIMIT=240` if never synced.
- **OAuth login URL (device flow):** `POST /cb/oauth/device/start` → `auth_url` + `state` (upstream `POST /v2/plugin/auth/state?platform=CLI`); poll `GET /cb/oauth/device/poll?state=` until ready → client imports via `/cb/oauth/import`. Dashboard Add OAuth modal: **Manual | Login URL**.
- **Credential probe (Test):** `POST /cb/keys/test` (key or email) and `POST /accounts/test` (Grok email). Hits upstream directly with that credential (CB `gpt-5.5`, Grok `grok-4.5`); not pool RR. Disabled credentials still probed.

### Grok aliases
`grok-4.5-{high,medium,low,xhigh,auto,none}` → `grok-4.5` + `reasoning_effort` (client wins if set).

## File map
| File | Role |
|------|------|
| `main.go` | Version, HTTP clients, workers, routes, middleware, graceful shutdown |
| `auth_adapter.go` | Type aliases + bridges to `internal/auth` (Manager, SessionStore, etc.) |
| `handlers_adapter.go` | Handler function wrappers (for signature-changed handlers) |
| `csrf_guard.go` | Origin/Referer check on cookie-authed mutations (P2-2) |
| `login_limiter.go` | IP-based rate limiter for `/login` (5/min + 20/hour) |
| `grok_account.go` | Grok pool, refresh, proxyGrok, reenableWorker |
| `codebuddy.go` | CB pool, transform, proxyCodeBuddy, reenableCBWorker |
| `proxy.go` | Routing, RequestLog build |
| `db.go` | Redis + LogStore glue (async batch pipeline, factory) |
| `internal/db/logstore.go` | `LogStore` interface + shared DTOs (RequestLog, RequestStats, …) |
| `internal/db/logstore_sqlite.go` | modernc.org/sqlite backend (default) |
| `internal/db/logstore_clickhouse.go` | ClickHouse backend (opt-in via `LOG_BACKEND=clickhouse`) |
| `handlers.go` | health, accounts, history, keys, dashboard static |
| `auth.go` / `ratelimit.go` / `health.go` | Auth, RPM, circuit |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rilspratama/Foxrouters](https://github.com/rilspratama/Foxrouters) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
