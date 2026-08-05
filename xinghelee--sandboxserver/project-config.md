---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## What this is

SandboxServer is a **DEBUG-only iOS SDK** that embeds an HTTP + WebSocket server inside a host
app. It serves a browser web console and a token-authenticated REST/WS API for browsing the
sandbox file system, databases, and live network traffic. The same on-device API is consumed by a
standalone MCP bridge so AI clients can drive it. **Zero third-party runtime dependencies** — the
transport is hand-rolled on Apple's Network.framework.

Three codebases live here:
- **Swift SDK** (`Package.swift`, `Sources/`) — the embedded server. The primary, validated artifact.
- **`web-src/`** — the Preact + TypeScript console (Vite). Build output is committed into
  `Sources/SandboxServerCore/Resources/web/` and served by the SDK. **Edit `web-src/`, never the
  committed `Resources/web/` output directly.**
- **`mcp-bridge/`** — the standalone `sandbox-mcp` npm package (its own module graph; not part of SPM).

## Commands

```bash
# Swift SDK — the real server is gated behind the SandboxServerEnabled trait:
swift build --traits SandboxServerEnabled                 # build the real core
swift test  --traits SandboxServerEnabled                 # run all tests (unit + end-to-end)
swift build                                               # build the Release-safe NO-OP path (no trait)

# Single test:
swift test --traits SandboxServerEnabled \
  --filter SandboxServerCoreTests.WebSocketCodecTests/testAcceptKeyMatchesRFCExample

# Web console:
cd web-src && npm install && npm run build                # → Sources/SandboxServerCore/Resources/web
VITE_API_BASE=http://<device-ip>:<port> npm run dev       # HMR proxied to a running device

# MCP bridge:
cd mcp-bridge && npm install && npm run build             # tsc → dist/

# iOS demos (xcodegen + simulator) — prove the SDK on a real iOS app end-to-end. Two live under
# Examples/: `Showcase` seeds every panel full of data; `Tasks` is a realistic app (organic data).
cd Examples/Showcase && ./run.sh                          # build → install → launch → open console URL
# manual: xcodegen generate; xcodebuild -project Examples/Showcase/SandboxShowcaseDemo.xcodeproj \
#   -scheme SandboxShowcaseDemo -destination 'platform=iOS Simulator,name=iPhone 17 Pro Max' build

# Run the server on macOS for quick browser testing:
swift run --traits SandboxServerEnabled SandboxServerDevHost
```

> Running `swift build`/`swift test` **without** `--traits SandboxServerEnabled` exercises the
> no-op path: `SandboxServerCore` is not in the dependency graph and all tests compile to empty
> (each test file is wrapped in `#if SandboxServerEnabled`). Both paths must stay green.

The package declares `.macOS(.v11)` in addition to `.iOS(.v14)` **solely so the package builds and
tests on a Mac host** (the end-to-end test boots a real server on loopback). iOS is the product
target; UIKit-only code is behind `#if canImport(UIKit)`.

## The one abstraction that matters: `SandboxPlugin`

The core (`SandboxServerCore`) knows nothing about files, databases, or networking. Every feature
is a `SandboxPlugin` (defined in `Sources/SandboxServerAPI/SandboxPlugin.swift`) mounted under
`/__sandbox/api/v1/<id>/`. A plugin's `capabilities` (`PluginCapabilities`) are aggregated into the
manifest at `GET /__sandbox/api/v1/plugins`, and **that single manifest drives both** which web
console panel renders (`panelKey`) **and** which MCP tools the bridge registers (`mcpTools`). To add
a feature, conform to `SandboxPlugin` and `register()` it — touch nothing in the transport/router.

Request/response flow: `NetworkFrameworkTransport` accepts a connection → `HTTPConnectionReader`
parses the request → `MiddlewareChain` (auth + DNS-rebinding host check) runs **before any plugin**
→ `Router.match` resolves the plugin route → the plugin handler returns an `SBResponse`. WebSocket
upgrades (`/__sandbox/ws`) are handed to the `WSHub` actor, which fans events to subscribers over a
single multiplexed connection (`{channel, type, seq, payload}`, `seq` monotonic per channel).

`PluginContext` is the only handle a plugin gets to the running server: `publish` (WS events),
`extraRoots`, `hostValue`, `config`, `log`. It deliberately does not expose the transport or hub.

## Wire contract (frozen — web console + MCP both depend on it)

- REST under `/__sandbox/api/v1`; success `{ "data": …, "meta": { apiVersion, ts } }`,
  error `{ "error": { code, message, details } }`, lists `{ data: { items, nextCursor } }`.
- `GET /healthz`, `GET /plugins` (manifest), then per-plugin routes under `/<id>/`.
- Network plugin is **live** (`/net/requests`, `/net/requests/{id}`, `DELETE /net/requests`,
  `net` WS channel). File plugin is **live** (`/fs/roots`, `/fs/list`, `/fs/stat`, `/fs/file`
  GET/PUT with Range, `/fs/move`, `DELETE /fs/file`) — every path is confined to an allowed root
  (app container + host-registered extra roots; traversal → 403). DB plugin is **live** (read-only):
  `GET /db` scans for SQLite files; `/db/{dbId}/tables`, `/db/{dbId}/tables/{table}/schema`, and
  `POST /db/{dbId}/query` (browse a table or run a SELECT) read via a `SQLITE_OPEN_READONLY`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xinghelee/SandboxServer](https://github.com/xinghelee/SandboxServer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
