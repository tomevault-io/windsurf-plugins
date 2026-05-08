---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

local.vibe — a local DNS daemon that gives dev servers friendly `.vibe` names. CLI binary is `vibe`. Single Go binary, minimal dependencies (only Cobra for CLI, rest is stdlib).

## Workflow

- **Never commit unless the user explicitly asks.** Do not auto-commit after completing a task or bundle commits into other actions.
- **Always run `vibe dev` after changes** to rebuild and restart the daemon.

## Build & test

```bash
vibe dev              # rebuild binary + restart daemon (do this after every change)
go build -o vibe .    # just build
go test ./...         # run tests
go vet ./...          # lint (also runs in CI)
```

The daemon runs a compiled binary at `/opt/homebrew/bin/vibe`, not source — changes aren't picked up until rebuilt. `vibe dev` handles the full cycle: build → install → kill old daemon → LaunchAgent auto-restarts with new binary.

## Architecture

**Request flow:** Browser → dnsmasq (*.vibe → 127.0.0.1) → pf (443 → 7443, 80 → 7999) → daemon (HTTPS or HTTP) → reverse proxy → app on target port. Bookmark routes either redirect (307) to an external URL or reverse-proxy to it (when `proxy=true`). HTTP requests redirect (301) to HTTPS when TLS is enabled.

**Three layers, strictly separated:**

1. **CLI commands** (`cmd/`) — Cobra commands. Know nothing about daemon internals. Use `internal/client` to talk to daemon.
2. **Client** (`internal/client/`) — HTTP wrapper. Tries Unix socket (`~/.vibe/vibe.sock`) first, falls back to TCP (`127.0.0.1:7999`).
3. **Daemon** (`internal/daemon/`) — HTTP server with embedded HTML dashboard. Core components:
   - `daemon.go` — Server struct, Start/Stop, HTTP routing by Host header, TLS listener with cert hot-reload
   - `api.go` — REST endpoints under `/_api/` (register, deregister, update, list, health, start, stop, ready, repair, preferences). Route names validated as DNS-safe (lowercase alphanumeric + hyphens). All user input HTML-escaped in dashboard output.
   - `routes.go` — Thread-safe RouteTable (RWMutex + map), RouteType enum, atomic per-route Failure diagnostics
   - `process.go` — ProcessManager spawns/kills managed child processes (uses process groups for clean shutdown). On immediate crash, returns a structured `StartError` with the tail of the log file.
   - `monitor.go` — Background goroutine sweeps dead PIDs and expired TTLs every 5s
   - `persistence.go` — Saves/loads sticky, managed, and bookmark routes to `~/.vibe/routes.json`
   - `dashboard.go` — Embedded HTML dashboard with modal UI for adding/editing routes
   - `startpage.go` — "Not running" page for stopped managed routes with Start button; surfaces recovery hints as a one-click "Kill PID X and Retry"
   - `repairpage.go` — "Reconnecting..." page shown when a route's port goes dark; polls `/_api/routes/{name}/repair` to auto-discover the new port
   - `log_scan.go` — regex patterns for extracting recovery hints (orphan PID, EADDRINUSE) from failed process log tails
   - `port_discover.go` — finds a managed route's real listening port via `lsof` on the process group and log-tail regex
   - `proxy_bookmark.go` — reverse-proxy for bookmark routes with `proxy=true` (landing path redirect, Location/cookie rewrites, X-Forwarded-For suppression)
   - `theme.go` — Shared CSS/HTML head (Geist fonts, Vercel-inspired dark theme)
   - `setup_md.go` — Markdown setup guide served at `/setup.md`

**Cert** (`internal/cert/`) — Generates local ECDSA CA + wildcard leaf certs using Go stdlib. Installs CA in macOS Keychain. Leaf certs use explicit SANs per route (Chrome rejects `*.vibe` wildcards).

**Config** (`internal/config/`) — Loads `~/.vibe/config.json`, falls back to defaults. Daemon port 7999, TLS port 7443 (disabled by default, enabled by `vibe setup`), TLD "vibe", log level "warn", dashboard view "list".

## Route types

Five route types with different lifecycle semantics:
- **sticky** — `vibe register`; persists across daemon restarts; reverse-proxied
- **pid** — API only; auto-removed when tracked PID dies
- **ttl** — `--ttl` flag on register; auto-expires after N seconds
- **managed** — `vibe start` (reads `vibe.json` or inline args); daemon manages the child process, dashboard has start/stop buttons. Port can be omitted for auto-assignment.
- **bookmark** — External URL (e.g. Tailscale address); persists across restarts; visiting `name.vibe` either 307-redirects to the external URL or reverse-proxies to it (per-route `proxy` flag). Added/edited via dashboard modal.

## Key patterns

- **Route status:** Two separate fields — `Running` (process is alive) and `Ready` (port is accepting TCP connections). Managed routes start `Running=true, Ready=false`; a background goroutine polls the port every 500ms for up to 30s and flips `Ready=true` once the port responds. This handles REPL-wrapped servers where the process is alive before the HTTP server binds.
- **Dual communication:** Unix socket (preferred) with TCP fallback. Same HTTP mux serves both.
- **Thread safety:** RouteTable uses RWMutex, ProcessManager uses Mutex.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [graiz/local.vibe](https://github.com/graiz/local.vibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
