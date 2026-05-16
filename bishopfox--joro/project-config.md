---
trigger: always_on
description: **Update this file whenever significant changes are made to the project** - new packages, changed commands, architectural decisions, or new conventions. Outdated instructions cause mistakes. If you add a dependency, change a build step, or restructure a package, update the relevant section here before finishing the task.
---

# Joro - Claude Code Instructions

## Keeping This File Fresh

**Update this file whenever significant changes are made to the project** - new packages, changed commands, architectural decisions, or new conventions. Outdated instructions cause mistakes. If you add a dependency, change a build step, or restructure a package, update the relevant section here before finishing the task.

---

## Project Overview

Joro is an intercepting HTTP/HTTPS proxy and web shell toolkit for penetration testing. It is a single Go binary that starts a proxy server and serves a React web UI - there is no CLI mode.

Three modes:
- **Proxy mode** (default): intercepting proxy + web UI
- **Listener mode** (`--listener`): out-of-band callback server (DNS + HTTP + SMTP) for blind vuln detection
- **Team Server mode** (`--listener --teamserver`): listener + authenticated team collaboration (chat, notes)

Ports & paths:
- Proxy `:8080` (`--proxy-port`), UI/API `:9090` (`--ui-port`)
- Data dir `~/.joro/` — CA cert/key + `callbacks.db`
- Listener: DNS `:53` (`--dns-port`), HTTP `:80` (`--http-port`), HTTPS `:443` (`--https-port`, `0` to disable), SMTP `:25` (`--smtp-port`, `0` to disable), SMTPS `:465` (`--smtps-port`, `0` to disable), domain via `--domain` or UI, optional external TLS cert via `--tls-cert` + `--tls-key` (both required; replaces the auto-generated self-signed leaf, shared by HTTPS and SMTPS/STARTTLS)

---

## Repository Structure

```
main.go                      Entrypoint (proxy or listener mode)
internal/
  config/                    Config struct + defaults
  event/                     Shared WSEvent struct (avoids proxy/callback import cycle)
  callback/                  SQLite (modernc.org/sqlite), token CRUD, DNS + HTTP listeners
  cert/                      ECDSA P-256 CA, leaf gen, sync.Map cache
  proxy/
    handler.go               ServeHTTP: CONNECT vs plain HTTP
    mitm.go                  TLS termination + HTTP/1.1 loop
    intercept.go             Per-request channel queue with timeout
    noise.go                 Silently tunnels browser background traffic
    scope.go                 Two-level scope (host + method/path)
    store.go                 Thread-safe ring buffer
    replace.go               Match & Replace (raw-byte rules)
    customdata.go            Additive header/query/body injection
    websocket.go ws_relay.go ws_store.go   WS MITM (custom frames over net.Conn)
    ws_manipulate.go         User-driven outbound WS sessions
    client.go helpers.go     HTTP client + utilities
  team/                      Team chat + notes tables, bearer-token auth middleware
  fuzzer/                    Goroutine-pool fuzzer + in-memory campaign store (max 50)
  shell/                     ASP/ASPX/PHP/etc. shell gen + executor + dictionary
  sliver/                    gRPC client for Sliver C2 (custom protowire encoding)
  plugins/                   Plugin lifecycle: load, categorize, init, shutdown
  api/
    server.go routes.go      APIServer + route registration + SPA embedding
    ws.go                    WebSocket hub (gorilla/websocket)
    handlers_*.go            Per-feature handlers (requests, intercept, manipulate,
                             generate, execute, fuzzer, settings, certs, callbacks,
                             replace, customdata, plugins, team, sliver, ws, ...)
    ws_relay.go              Relay to teamserver, forwards team.* events
sdk/sdk.go                   Plugin SDK: interfaces, types, constants (separate Go module)
web/
  embed.go                   //go:embed dist
  dist/                      Built frontend (gitignored except placeholder)
  src/
    main.tsx App.tsx index.css vite-env.d.ts
    themes/bishop-fox.css    Default dark theme (BF brand palette)
    lib/api.ts ws.ts         Typed fetch wrapper + WS singleton (auto-reconnect)
    stores/*.ts              Zustand: request, fuzz, intercept, settings, callback,
                             ws, manipulateWS, team
    pages/                   History, Intercept, Manipulate (HTTP+WS), Generator,
                             Executor, Fuzz, Login, Settings, Callbacks, Plugins,
                             PluginTabPage
    components/              DynamicConfigForm (auto-gen plugin ExecProvider config)
examples/plugins/
  hello-provider/            ExecProvider + GraphProvider example
  hello-tab/ hello-feature/  Top-level tab + sub-tab plugin examples
  hello-dashboard/           Dashboard replacement example
  interactsh/                InteractProvider: stdlib-only interactsh client
Makefile                     build, build-frontend, build-all, dev, clean
```

---

## Build Commands

```bash
go build ./...               # Go-only (uses placeholder frontend, works without npm)
make build                   # Full (frontend + Go binary)
make build-all               # Cross-platform → dist/
make dev                     # Backend with --dev flag (proxies UI to Vite)
cd web && npm run dev        # Vite dev server (separate terminal, with `make dev`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BishopFox/joro](https://github.com/BishopFox/joro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
