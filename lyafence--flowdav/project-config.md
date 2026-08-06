---
trigger: always_on
description: | `make build` | Build unified binary → `./bin/flowdav` |
---

# flowdav — Agent Instructions

## Commands

| Command | What |
|---------|------|
| `make build` | Build unified binary → `./bin/flowdav` |
| `make check` | Full verification: vet → lint → build → test (with race detector) |
| `make test` | Unit tests with race detector (`-timeout 120s`) |
| `make test-short` | Unit tests without race detector |
| `make bench` | Run benchmarks: `go test -bench=. -benchmem -timeout 180s ./internal/transport/` |
| `make test-e2e` | E2E tests (requires podman; runs `scripts/test_e2e.sh`) |
| `make test-e2e-encrypted` | E2E + encrypted configs |
| `make fuzz` | Run fuzz tests (30s each: transport envelope) |
| `make vulncheck` | Run govulncheck vulnerability scan |
| `make encrypt FILE=config.json` | Encrypt config (also: `FLOWDAV_PASSWORD=secret make encrypt`) |
| `make tidy` | `go mod tidy -e` (CI checks `git diff go.mod go.sum`) |
| `make nuke` | Full env reset (compose down, clean images, build artifacts) |
| `make android-apk` | Build Android APK (debug) → `bin/flowdav-android.apk` |
| `make compose-android` | Build Docker images for Android test env |
| `make android-deploy` | Build APK + start test env + deploy to Android device. **Requires `make compose-android` first.** |

> **Container tool:** Makefile targets use `podman` (`docker-build`, `image-to-bin`, `compose-*`). Do not assume `docker`.

## Package Map

| Package | Responsibility |
|---------|---------------|
| `cmd/flowdav` | Entrypoints (thin) — unified binary |
| `cmd/android` | Gomobile bridge (exported to Android) |
| `internal/config` | Load, validate, encrypt/decrypt configs |
| `internal/transport` | Engine (poll loop, sessions), Envelope, Crypto, VirtualConn (SOCKS5), Pool |
| `internal/storage` | WebDAV backend + MultiBackend (circuit breaker, round-robin, 429-aware cooldown) |
| `internal/logger` | Leveled logging |

## Architecture

```
SOCKS5 ←→ client ←→ WebDAV ←→ server ←→ destination
```

**Binary:** `flowdav` — unified entrypoint with `-c` (client), `-s` (server), `-e` (encrypt), `-g` (generate config) modes.  
**Android bridge:** `cmd/android/bridge.go` — gomobile bind, exports `StartProxyFromData`/`StartProxyManual`/`StopProxy`/`GetStatus`/`StopAndError`/`SetSocks5Auth` to Kotlin. **Global state intentional** (gomobile exports free functions, not objects). **Config validation delegates** to `config.ValidateAppConfig` — the two paths share the same validation logic. See `parseConfigJSON` in bridge.go. **`GetStatus` violates "no getters" intentionally** — Java/Kotlin interop dictates Java Beans naming.

## Design Invariants

- Server has **zero listening ports** for data — all data via WebDAV storage (optional health endpoint on loopback only).
- WebDAV is passive dumb storage; client encrypts/muxes, server decrypts/demuxes.
- AES-256-GCM + HMAC-SHA256 on all data.
- Random filenames `{dir_byte}{16_hex}` — no metadata leaks. Mapped to `{subdir}/{uppercase_hex}` on storage (direction byte → subdirectory: `r`→`invoices`, `s`→`receipts`).
- DNS leak protection: raw resolver, UDP explicitly blocked.
- Multi-WebDAV: random session assignment, round-robin upload fallback.
- **429-aware circuit breaker**: rate-limit (429) uses separate 60s cooldown without incrementing failure count. Session auto-migrates to another backend on 429. Download falls back to non-indexed search across all backends.
- **TLS fingerprint masking**: uTLS `HelloChrome_133` by default (configurable via `tls_fingerprint`). User-Agent matches (`Chrome/133.0.0.0`).
- No global state (exceptions: `transport.MaxMessageSize` / `storage.MaxFileSize` — package-level vars set at startup, justified by OOM prevention per `internal/transport/crypto.go` `MaxMessageSize` var; `internal/logger` — package-level `level` var for leveled logging is standard and excluded by convention; `cmd/android/bridge.go` — gomobile requires free functions, not objects, so global state is unavoidable and documented as intentional).
- **Operational philosophy:** minimize external observability, avoid predictable patterns. When adding anything network-facing: is it optional? bounded? indistinguishable from noise?

## Pre-commit Hook

The pre-commit hook (`.githooks/pre-commit`) checks:
- `gofmt` on `cmd/ internal/`
- `goimports` with `-local github.com/lyafence/flowdav`
- `go vet ./...`
- Bans `math/rand`, `os/exec` in production code (`cmd/`, `internal/`, excludes `_test.go`)
- Bans `database/sql` in production code (same scope)
- Bans `sync.Pool` in production code (same scope)

Install with `make hooks`.

## Config Quick Reference

- Flags: `-c config.json` (client), `-s config.json` (server), `-e config.json` (encrypt), `-g config.json` (generate), `-p master_password`, `-l loglevel`, `--version`
- Fields: `enc_key` / `hmac_key` (32-byte base64), `max_message_size` (default 16MB), `max_sessions` (default 0 = unlimited), `idle_timeout_ms` (default 10000), `webdav.backends` (array), `health_port` (e.g. `"127.0.0.1:9191"`), `log_level` (`debug`, `info`, `warn`, `error`)
- `padding_size` (bucket size for tail padding; 0 = off), `hold_ms` (max server-side random delay before flush; 0 = off) — optional settings
- Health: `GET /health` on `health_port` → JSON stats (active sessions, retry counters, tx queue depth, per-backend circuit breaker state)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lyafence/flowdav](https://github.com/lyafence/flowdav) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
