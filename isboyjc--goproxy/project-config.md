---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GoProxy is an intelligent proxy pool system written in Go. It automatically fetches HTTP/SOCKS5 proxies from public sources, validates them (exit IP + geolocation + latency), and serves them via 4 proxy ports (HTTP random/stable, SOCKS5 random/stable) plus a WebUI dashboard.

## Build & Run

```bash
# Run directly (requires Go 1.25, CGO enabled for sqlite3)
go run .

# Build and run
go build -o proxygo .
./proxygo

# Docker
docker compose up -d
```

CGO is required (`CGO_ENABLED=1`) because of the `github.com/mattn/go-sqlite3` dependency.

## Testing

There are no Go unit tests (`go test ./...`). Testing is done via shell scripts against a running instance:

```bash
# HTTP proxy test (continuous, Ctrl+C to stop)
./test/test_proxy.sh          # port 7777 (random)
./test/test_proxy.sh 7776     # port 7776 (stable)

# HTTP proxy HTTPS access test (random visits to Google/OpenAI/GitHub etc.)
./test/test_http_https.sh           # port 7777, continuous
./test/test_http_https.sh 7776 20   # port 7776, 20 iterations

# SOCKS5 proxy test
./test/test_socks5.sh localhost 7779      # random
./test/test_socks5.sh localhost 7780 50   # stable, 50 iterations

# Go/Python test scripts
go run test/test_proxy.go 7777
python test/test_proxy.py 7776
```

## Architecture

The system is a single binary with several cooperating goroutines. Module `go.mod` name is `goproxy`.

### Module Dependency Flow

```
main.go (orchestrator)
  ├── config/     — Global config (env vars + config.json), thread-safe singleton
  ├── storage/    — SQLite persistence layer (proxies + source_status tables)
  ├── fetcher/    — Multi-source proxy fetcher with circuit breaker (SourceManager)
  ├── validator/  — Concurrent proxy validation (connectivity + exit IP + geo + latency)
  ├── pool/       — Pool manager (admission control, slot allocation, replacement logic)
  ├── checker/    — Background health checker (batch-based, skips S-grade when healthy)
  ├── optimizer/  — Background quality optimizer (replaces slow proxies with faster ones)
  ├── custom/     — Custom proxy subscription manager (fetch, parse, validate, periodic refresh)
  │   ├── parser.go   — Clash YAML / plain / base64 subscription parser
  │   ├── singbox.go  — sing-box process manager (config generation, start/stop/reload)
  │   └── manager.go  — Subscription refresh loop + probe-wake loop for disabled proxies
  ├── proxy/      — Outward-facing proxy servers
  │   ├── server.go       — HTTP proxy (implements http.Handler)
  │   └── socks5_server.go — SOCKS5 proxy (raw TCP, manual protocol implementation)
  ├── webui/      — Dashboard server (embedded HTML in html.go, API in dashboard.go)
  └── logger/     — In-memory log collector for WebUI display
```

### Key Design Patterns

- **Pool state machine**: healthy → warning → critical → emergency. State determines fetch mode (optimize/refill/emergency) and latency thresholds.
- **Slot-based capacity**: Pool has fixed size split between HTTP/SOCKS5 by configurable ratio (default 3:7). Each protocol has guaranteed minimum slots.
- **Smart admission**: New proxies enter if slots available, or replace worst existing proxy if significantly faster (30%+ by default via `ReplaceThreshold`). HTTP proxies must also pass an HTTPS CONNECT tunnel test (random real HTTPS site visit with retry) before admission. Geo-filter (whitelist/blacklist) is applied during validation before admission.
- **Geo-filter**: Whitelist (`AllowedCountries`) takes priority — if non-empty, only those countries pass. Otherwise blacklist (`BlockedCountries`) rejects listed countries. Configurable via env vars, `config.json`, or WebUI at runtime. On startup, existing proxies violating the filter are cleaned from the database.
- **Protocol-parallel validation**: `smartFetchAndFill` splits candidates by protocol and validates SOCKS5/HTTP concurrently. SOCKS5 fills faster (no HTTPS check overhead); HTTP validation runs in parallel without blocking SOCKS5 admission.
- **Circuit breaker on sources**: `SourceManager` tracks consecutive failures per source URL. 3 fails → degraded, 5 → disabled for 30min.
- **Auto-retry on proxy failure**: Both HTTP and SOCKS5 servers retry with different upstream proxies on failure (up to `MaxRetry` times), deleting failed proxies immediately.
- **SOCKS5 service only uses SOCKS5 upstreams** (many free HTTP proxies don't support CONNECT). HTTP service can use either protocol upstream.

### Background Goroutines (started in main.go)

1. **Status monitor** — every 30s, checks pool state and triggers `smartFetchAndFill` if needed
2. **Health checker** — every `HealthCheckInterval` min, validates a batch of proxies
3. **Optimizer** — every `OptimizeInterval` min, fetches from slow sources and replaces B/C grade proxies
4. **Config watcher** — listens for WebUI config changes and adjusts pool slots

### Ports

| Port | Service |
|------|---------|
| 7776 | HTTP proxy (lowest-latency mode) |
| 7777 | HTTP proxy (random rotation mode) |
| 7778 | WebUI dashboard |
| 7779 | SOCKS5 proxy (random rotation mode) |
| 7780 | SOCKS5 proxy (lowest-latency mode) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isboyjc/GoProxy](https://github.com/isboyjc/GoProxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
