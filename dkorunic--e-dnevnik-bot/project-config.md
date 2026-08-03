---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

For a full tour of components, data flow, and design trade-offs, read **`ARCHITECTURE.md`**. This file only documents the things not easily discovered from the code.

---

## Package layout

All sub-packages live under `internal/` (enforced by the Go toolchain — nothing outside this module can import them):

| Package | Role |
|---|---|
| `internal/msgtypes` | Canonical domain event: `Message` struct + `EventCode` enum. No deps. |
| `internal/fetch` | Raw HTTP client for e-Dnevnik (SAML/SSO auth, cookie jar). |
| `internal/scrape` | Parses `fetch/` HTML into `msgtypes.Message` events. |
| `internal/sqlitedb` | SQLite KV dedup store. |
| `internal/codec` | CBOR (`fxamacker/cbor/v2`) encode/decode for `[]Message` queue persistence. |
| `internal/queue` | Dead-letter queue built on `sqlitedb` + `codec`. |
| `internal/messenger` | Six messenger backends (Discord/Telegram/Slack/Mail/Calendar/WhatsApp). |
| `internal/format` | Plain/HTML/Markdown formatters consumed by messengers. |
| `internal/oauth` | Google Calendar OAuth2 interactive flow (local HTTP server). |
| `internal/config` | TOML config load + validation. |
| `internal/logger` | Global `zerolog` wrapper. |
| `internal/version` | Reads dependency version from binary build info. |

Root-level files in the `main` package:

| File | Responsibility |
|---|---|
| `main.go` | Entry point, polling ticker loop, goroutine lifecycle, PGO/profiling. |
| `routines.go` | `scrapers`, `msgDedup`, `msgSend`, `versionCheck` — the three pipeline stages. |
| `init.go` | Interactive first-run setup for WhatsApp (pairing) and Calendar (OAuth2). |
| `flags.go` | All CLI flags via `peterbourgon/ff/v4`. Flag vars are **package-level pointers** (see below). |
| `db.go` | `openDB` / `closeDB` helpers. |
| `log.go` | `initLog` — wires log level and colorized/JSON output from flag vars. |

---

## Toolchain

- **Go 1.26+ is mandatory** — the code calls `sync.WaitGroup.Go` (Go 1.25) and `context.WithoutCancel` (Go 1.21). `go.mod` pins `go 1.26`; older local toolchains will trigger an auto-download via `GOTOOLCHAIN` or fail to build.
- Build system: [Task](https://taskfile.dev/) via `Taskfile.yml`. `CGO_ENABLED=0` is set at the taskfile level; do not override — the whole point of `modernc.org/sqlite` is a static binary.

## Commands

```bash
task build          # fmt → static PGO build with ldflags (CGO off)
task build-debug    # fmt → race build (CGO on). Slower. Use for races only.
task test           # go test ./...
task lint           # fmt → golangci-lint (5m timeout)
task lint-nil       # fmt → nilaway. Separate pass, NOT part of `task lint`.
task fmt            # go mod tidy + gci + gofumpt + betteralign -apply
task modernize      # apply gopls/modernize fixes across the tree
task update         # go get -u + go mod tidy
task update-tools   # install gci, gofumpt, betteralign (required for `task fmt`)
task tools          # verify the three formatters are on PATH
```

Single test: `go test -run TestName ./path/to/package/`.

The main binary accepts `-t`/`--test` for an **emulation mode** that pushes a synthetic message through the full pipeline without scraping — use this to verify messenger credentials and formatting without waiting for real events.

`-0` / `--fulldebug` logs every scraped event before the dedup filter — the fastest way to debug "why didn't this alert fire?" questions. Implies `-v`.

`-c <file>` / `-m <file>` write CPU and heap pprof profiles. The production build uses `-pgo=auto`; the toolchain picks up `default.pgo` from the repo root automatically.

## Mandatory before every commit

`task fmt` must pass. It runs three tools, **all of which must be installed** (`task update-tools`):

1. `gci` — import ordering
2. `gofumpt` — stricter gofmt
3. `betteralign` — struct field alignment

`betteralign -apply` rewrites struct field order to minimise padding. It will reorder fields in types you touch — this is expected, do not revert it.

---

## Load-bearing invariants

These behaviours are not enforced by the type system or tests. Breaking them manifests as data loss, duplicate alerts, or shutdown hangs that only appear in production.

### Shutdown-tolerant queue writes — `internal/messenger/common.go`

Every messenger that stores un-delivered messages to the retry queue uses `queueStoreCtx(ctx)` — built on `context.WithoutCancel` + `storeTimeout = 5s`. The purpose: when the main context is already cancelled (SIGTERM arrived mid-send), the final sqlite write to the failed-message queue **must still complete**, otherwise the message is lost forever.

When adding a new messenger or touching send paths, the post-send `StoreFailedMsgs` call must use `queueStoreCtx`, not the raw `ctx`.

### Two-level WaitGroup in `msgSend` — `routines.go`

`msgSend` uses a dedicated `wgInner` to track the per-messenger goroutines. The deferred sequence closes **every messenger channel** first, **then** `wgInner.Wait()`. Reversed ordering deadlocks because each messenger's `range` loop only exits once its channel is closed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dkorunic/e-dnevnik-bot](https://github.com/dkorunic/e-dnevnik-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
