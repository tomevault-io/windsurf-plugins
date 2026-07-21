---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`api-key-rotator` is a Go reverse proxy that provides key rotation for
multiple API providers, currently **Firecrawl** and **Tavily**. Each provider
is a "profile" with its own key pool, upstream base URL, route prefix, and
rotation policy.

- **Firecrawl**: requests without a Tavily prefix go to `api.firecrawl.dev`,
  with keys selected by remaining credits. Crawl `next` pagination URLs are
  rewritten so pagination stays under rotation.
- **Tavily**: requests with `/tavily` prefix are routed to `api.tavily.com`
  (prefix stripped). Rotation is purely status-code-driven (no body denylist).

The whole point: point firecrawl-mcp's `FIRECRAWL_API_URL` at this proxy and
get key rotation with **zero changes** to firecrawl-mcp. Tavily works by
sed-replacing the Tavily API URL inside OpenWebUI at container startup.

Stdlib-only (`go.mod` has no dependencies), single binary, no external state.

## Commands

```bash
go test ./...                      # run all tests (no Go on host: use docker)
go test -run TestRotator_RotatesOn402 ./...   # single test
go test -run TestRotator ./...      # all tests matching a pattern
go vet ./...
go build -o api-key-rotator .       # build binary

FIRECRAWL_API_KEYS=fc-x ./api-key-rotator   # run locally (firecrawl only)
./api-key-rotator -healthcheck     # GET /healthz on 127.0.0.1:PORT, exit 0/1 (Docker healthcheck)

# Docker-based test (no Go on host):
docker run --rm -v "$PWD":/src -w /src golang:1.26-alpine sh -c "go test ./... && go vet ./... && go build -o /tmp/api-key-rotator ."
docker build -t api-key-rotator:test .

docker compose up -d --build        # run via compose (rotator + reference mcpo+firecrawl-mcp)
```

CI (`.github/workflows/build-docker.yml`) builds and pushes `ghcr.io/<repo>:latest` + `:sha` on push to main when `Dockerfile`, `*.go`, `go.mod`, or the workflow change.

## Architecture

Request flow lives in `proxy.go`'s `rotator.ServeHTTP`. Everything else exists to support it:

1. **Profile routing** (`profile.go`): `ServeHTTP` checks the request path
   against each profile's `RoutePrefix`. Non-matching requests are handled by
   the first (Firecrawl) profile. The matching profile's key pool, upstream,
   and rotation policy are used for the request.
2. **Buffer request body once** (`io.ReadAll`), then replay it across retry
   attempts - requests are not idempotent-safe to re-send, so the body must
   be re-readable.
3. **Rotation loop** (`maxRotations = MaxPasses * poolSize`): pick the best
   key via `pool.Current()` (highest `remainingCredits` above the stop
   threshold, skipping cooled-down keys). Call `tryKey`:
   - `tryKey` sends the request and, on a **transient** error
     (`shouldRetry`: network error, 403, 408, 5xx), retries the **same key**
     with exponential backoff `500ms/1s/2s/4s/8s` before returning. It only
     signals "give up on this key" (`netErr=true`) after backoff is exhausted.
   - **Over `MAX_BODY_BYTES`** -> forward untouched, break (no rotate, no rewrite).
   - **`shouldRotate`** (provider-dependent) -> record rejection, disable if
     `isCreditExhausted`, `Advance` (cools the key down ~30s), retry next key.
   - **Otherwise** -> record success, `Decrement` predicted credits by
     `creditsUsed` (or 1), trigger `MaybeRefreshLow`, break.
4. **After loop**: if no usable key, return `503`; if JSON and Firecrawl
   profile, rewrite `next` URLs (`rewrite.go`).

Key files and their roles:

| File | Responsibility |
|------|----------------|
| `main.go` | `buildServer` wires `Config` -> `[]Profile` -> transports -> clients -> `Refresher` per profile -> `rotator`. Routes `/healthz`, `/status`, `/`. `--healthcheck` flag. Starts goroutines: `RefreshAll` warm-up, `resetLoop`, `dailyRefreshLoop`. |
| `config.go` | `LoadConfig` parses all env vars; `buildProfiles` constructs Firecrawl + (optional) Tavily profiles. Validates thresholds (stop <= low). |
| `keys.go` | `KeyPool` - per-key `stats`, `disabled`/`disabledUntil`, `remainingCredits` (MaxInt64 = unmeasured), `cooldownUntil`. `Current`/`currentLocked` pick highest-credit usable key, skipping cooled-down keys (fallback to them if all cooled). `Advance` cools the current key ~30s so equal-credit keys actually rotate. `Decrement`/`SetCredits` adjust predicted/real balances. `AnyUsable` checks >= stop threshold. `Snapshot` masks keys + reports `remainingCredits` (-1 = unmeasured). |
| `profile.go` | `Profile` struct (pool, upstream, route prefix, rotation policy funcs). `matchProfile` routes requests. `getRotateFunc`/`getRetryFunc`/`getCreditExhaustedFunc` per profile. |
| `proxy.go` | The rotator: profile routing, rotation loop + `tryKey` (backoff retries on transient), header copying, body cap, disable-on-credit-exhaustion, credit decrement on success, rewrite+guard, 503 when no usable key. `backoffSchedule`, `extractCreditsUsed`, `readCapped`, `writeRawResponse`, `isHopByHop`. |
| `rotate.go` | `shouldRotate` / `shouldRetry` / `isCreditExhausted` — now profile-aware wrappers that dispatch to Firecrawl-specific or Tavily-specific logic. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xyonium/api-key-rotator](https://github.com/xyonium/api-key-rotator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
