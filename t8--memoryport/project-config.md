---
trigger: always_on
description: Memoryport is a Rust workspace that gives LLM interactions persistent, queryable memory using Arweave for permanent storage and LanceDB for local vector indexing. Data on Arweave is encrypted (AES-256-GCM) with per-batch keys and an Argon2id-derived master key. Includes a React dashboard (Tauri desktop + web), transparent API proxy for Anthropic/OpenAI/Ollama, and MCP server for Claude Code/Cursor.
---

# CLAUDE.md

## Project Overview

Memoryport is a Rust workspace that gives LLM interactions persistent, queryable memory using Arweave for permanent storage and LanceDB for local vector indexing. Data on Arweave is encrypted (AES-256-GCM) with per-batch keys and an Argon2id-derived master key. Includes a React dashboard (Tauri desktop + web), transparent API proxy for Anthropic/OpenAI/Ollama, and MCP server for Claude Code/Cursor.

**Website:** [memoryport.ai](https://memoryport.ai) — Next.js app in separate repo (`t8/memoryport-web`)
**AMP Spec:** [github.com/t8/amp-spec](https://github.com/t8/amp-spec) — open protocol for memory injection

## Build & Test

```bash
cargo check                # type-check entire workspace
cargo build                # build all crates
cargo test --workspace     # run all tests (167+ tests)
cargo build -p uc-cli      # CLI only
cargo build -p uc-mcp      # MCP server only
cargo build -p uc-proxy    # API proxy only
cargo build -p uc-server   # hosted API server only
cd ui && pnpm dev          # React dashboard dev server
```

### Dev Script (recommended)

```bash
./dev.sh start    # Build and start server + proxy + UI
./dev.sh stop     # Stop all services
./dev.sh restart  # Rebuild and restart everything
./dev.sh status   # Show what's running
./dev.sh server   # Rebuild and restart just the server
./dev.sh proxy    # Rebuild and restart just the proxy
./dev.sh ui       # Restart just the UI
./dev.sh logs proxy  # Tail proxy logs
```

### Tauri Desktop App

```bash
cd crates/uc-tauri/src-tauri && cargo tauri dev    # Dev mode (needs UI dev server running)
cd crates/uc-tauri/src-tauri && cargo tauri build   # Production .dmg/.app
```

### Prerequisites

- Rust stable (1.91+)
- `protoc` (Protocol Buffers compiler) — required by LanceDB's lance-encoding
  - macOS: `brew install protobuf`
- Node.js 18+ and pnpm (for UI)

### Binaries

| Binary | Crate | Purpose |
|--------|-------|---------|
| `uc` | uc-cli | CLI: init, store, query, retrieve, delete, rebuild-index, proxy, status, flush |
| `uc-mcp` | uc-mcp | MCP server (stdio) for Claude Code / Cursor |
| `uc-proxy` | uc-proxy | Multi-protocol API proxy (Anthropic + OpenAI + Ollama) |
| `uc-server` | uc-server | Multi-tenant hosted API server with auth, metrics, dashboard |

### Running the Stack Locally

```bash
# 1. Start the API server (port 8090 if Open WebUI uses 8080)
UC_SERVER_LISTEN=127.0.0.1:8090 ./target/debug/uc-server --config ~/.memoryport/uc.toml

# 2. Start the proxy (port 9191)
./target/debug/uc-proxy --config ~/.memoryport/uc.toml --listen 127.0.0.1:9191

# 3. Start the React dashboard (port 5174, proxies API to 8090)
cd ui && pnpm dev
```

## Architecture

9 crates in the workspace:

```
uc-arweave     — Arweave client (JWK wallet, ANS-104 data items, Turbo uploads, GraphQL)
uc-embeddings  — Embedding + LLM providers (OpenAI, Ollama) for embeddings and query enhancement
uc-core        — Core engine: chunker, tagger, batcher, writer, LanceDB index (chunks + facts tables),
                 retriever (hybrid vector+BM25+entity with RRF fusion), reranker, assembler,
                 analyzer, enhancer (query expansion + HyDE), facts (NLP extraction),
                 entities (Jaro-Winkler dedup), profile (user profile cache),
                 contradiction (fact superseding), crypto, keystore,
                 gate (3-gate retrieval gating), analytics, rebuild
uc-cli         — CLI binary with interactive setup wizard (`uc init`)
uc-mcp         — MCP server: 7 tools + 2 resources, auto-capture via uc_auto_store
uc-proxy       — Multi-protocol proxy: Anthropic /v1/messages, OpenAI /v1/chat/completions,
                 Ollama /api/chat + /api/generate. Context injection + auto-capture.
uc-server      — Multi-tenant HTTP API: per-user engine pool, API key auth, rate limiting,
                 Prometheus metrics, integration management, serves React dashboard
uc-tauri       — Tauri 2.x desktop app with sidecar binaries, lazy engine init, service manager,
                 auto-updater (checks GitHub releases for latest.json)
```

Frontend: `ui/` — React 19 + Vite + Tailwind. Dashboard, analytics, integrations, settings.

**Dependency graph:** `uc-arweave` and `uc-embeddings` have no internal deps. `uc-core` depends on both. Everything else depends on `uc-core`.

## Release & Distribution

### CLI Install
- **Canonical URL:** `curl -fsSL https://memoryport.ai/install | sh` (served by memoryport-web `src/app/install/route.ts`)
- **Repo script:** `install.sh` (installs to `~/.memoryport/bin`)
- Both install all 4 binaries: `uc`, `uc-mcp`, `uc-proxy`, `uc-server`

### Desktop App
- Built by `.github/workflows/tauri-build.yml` on version tags (`v*`)
- Platforms: macOS (aarch64, x86_64), Linux (x86_64). Windows disabled for now.
- macOS builds are code-signed and notarized (Apple Developer ID: Not Community Labs Inc)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [t8/memoryport](https://github.com/t8/memoryport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
