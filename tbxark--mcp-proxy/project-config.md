---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

An MCP proxy server (Go, single `main` package, flat layout) that aggregates multiple downstream MCP servers behind one HTTP entrypoint. Each configured downstream server is exposed at its own route (`<baseURL path>/<serverName>/`) via SSE or streamable HTTP.

## Commands

```bash
make build                      # CGO_ENABLED=0 build into ./build/ with version ldflags
go test ./...                   # run all tests
go test -short ./...            # unit tests only (skips the process-spawning end-to-end tests)
go test -run TestName .         # run a single test (all code is in the root package)
make format                     # full pipeline: go fix/fmt/vet, tests, go mod tidy, golangci-lint fmt+run, nilaway
./build/mcp-proxy --config config.json          # run the server
./build/mcp-proxy -check-config --config ...    # validate config and exit
```

`make format` expects `golangci-lint` and `nilaway` to be installed. Run it (or at least `go vet` + tests) before considering a change done.

## Architecture

Request flow: MCP client → HTTP mux (`http.go`) → per-server middleware chain (recover, optional logging, optional bearer-token auth) → `server.MCPServer` (mcp-go) → proxied downstream client (`client.go`).

- `main.go` — flag parsing and dispatch to one of three modes: normal server, `-authorize` (interactive OAuth), or `-auth-status`/`-doctor` (credential diagnostics).
- `config.go` — the (only) V2 config schema, validation, and loading via `confstore` (supports local file or HTTP(S) URL, with env-var expansion). Per-server `Options` inherit from `mcpProxy.options` defaults during `load()`. The V1 schema was removed; `FullConfig` still carries the old `server`/`clients` keys as `json.RawMessage` purely so a V1 file fails with a migration hint instead of "mcpProxy is required".
- `client.go` — wraps mcp-go clients for the three transports (`stdio`, `sse`, `streamable-http`). On startup, each client connects, initializes, and copies the downstream server's tools/prompts/resources/resource-templates onto the local `MCPServer` (with pagination and tool allow/block filtering). Every client gets a keepalive probe loop (`options.pingInterval`, default 30s) that drives the health state behind `/_readyz` — a real `tools/list` request on modern protocol connections (2026-07-28 removed ping), ping on legacy ones. `Client.connect` builds the transport, swaps it in under `mu`, and closes the previous one; tool/prompt/resource handlers resolve the live transport through `getClient()` at call time, so reconnecting is transparent to in-flight requests. `Start` gets the long-lived context — mcp-go's SSE transport ties its event stream to it and stdio stores it for request handling — so only `Initialize` is bounded by `connectTimeout`.
- `oauth.go` / `oauth_store.go` — OAuth client support for downstream servers requiring interactive auth (e.g. Notion). Tokens and dynamically-registered client credentials persist as JSON under the user config dir (`~/.config/mcp-proxy/oauth/` on Linux) via `FileTokenStore`, so a one-off `-authorize` run shares state with the daemon. `oauthAwareError` decorates connection failures with re-auth hints.
- `doctor.go` — backs `-auth-status` (local-only: config + cached token expiry) and `-doctor` (additionally connects to each remote server; may refresh tokens but never opens a browser).

Key behaviors to preserve:

- HTTP clients force `Accept-Encoding: identity` (`mcpHTTPHeaders` in `client.go`) — some MCP servers otherwise send gzip that mcp-go's JSON decoder can't handle.
- `/_healthz` and `/_readyz` are unauthenticated. `/_healthz` is process liveness; `/_readyz` is 503 until every client mounts, and 503 again (`status: degraded`) once a client that had connected fails `pingFailureThreshold` keepalive probes in a row — one missed probe is not enough, a busy single-threaded downstream simply doesn't answer while it works. Clients that never connected are excluded, so one broken server does not keep the proxy out of rotation; but if *nothing* mounted, readiness is 503 `status: unavailable`, since every route would 404.
- A failed downstream connection is logged but non-fatal unless that server sets `panicIfInvalid`. This covers client *creation* too: a stdio client spawns its subprocess in `newMCPClient`, so a missing command must not abort startup for everyone (`clientStartupError` in `http.go` is the single policy point).
- `autoReconnect` (per-server, default off) is opt-in self-healing, with two distinct timings: the startup goroutine in `http.go` retries an unreachable backend every `reconnectInterval` (default 15s) until it connects — mounting the route only then — while a connection that drops after being healthy is rebuilt by the ping task once it fails `pingFailureThreshold` probes in a row, so *its* recovery is paced by `pingInterval`, not `reconnectInterval`. `panicIfInvalid` still fails fast and is checked first, so retrying never overrides it. The route is published only after a successful connect, so readiness keeps treating a never-connected backend as unmounted.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TBXark/mcp-proxy](https://github.com/TBXark/mcp-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-27 -->
