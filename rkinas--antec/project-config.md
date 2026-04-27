---
trigger: always_on
description: > This file provides project context and coding standards for any AI coding assistant (OpenAI Codex, Cursor, Windsurf, Copilot Workspace, Cline, Aider, etc.). For Claude Code-specific instructions, see `CLAUDE.md`.
---

# Antec — AI Agent Development Instructions

> This file provides project context and coding standards for any AI coding assistant (OpenAI Codex, Cursor, Windsurf, Copilot Workspace, Cline, Aider, etc.). For Claude Code-specific instructions, see `CLAUDE.md`.

---

## Project Identity

**Antec** is a self-hosted personal AI assistant. Single Rust binary, zero mandatory dependencies, all data local. 15-crate Cargo workspace under `crates/`. Full PRD in `prd/` (30 documents).

**Read `prd/01-ARCHITECTURE.md` before writing any code.**

---

## Mandatory Checks

Run after EVERY change:

```bash
cargo build --workspace --lib
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all -- --check
```

All four must pass before any commit. No exceptions.

---

## Tech Stack

| Layer | Choice |
|-------|--------|
| Language | Rust 2021 edition, async via Tokio 1.x |
| HTTP/WS | Axum (Tower-based) — REST + WebSocket + SSE |
| Database | SQLite 3 (rusqlite, bundled) — WAL mode, FTS5, r2d2 pool (8 conns) |
| Config | TOML (human) + JSON (API) — 5-layer precedence |
| WASM Sandbox | Wasmtime — fuel metering + epoch interrupts |
| Serialization | serde + serde_json; MessagePack for internal wire |
| Crypto | ChaCha20-Poly1305 secret vault, HMAC-SHA256 audit chain |
| CLI | clap |
| Frontend | Vanilla HTML/CSS/JS (ES modules), embedded via rust-embed |
| i18n | Compile-time `t!()` macro — EN + PL |

---

## Crate Map

```
antec (workspace root)
 ├── src/main.rs                     # Binary entrypoint, CLI, boot
 └── crates/
      ├── antec-core/                # Agent loop, LLM providers, routing
      ├── antec-gateway/             # Axum HTTP/WS, REST routes, auth
      ├── antec-channels/            # Discord, WhatsApp, iMessage, Console
      ├── antec-tools/               # 50+ tools, risk classification
      ├── antec-memory/              # FTS5, BM25 + TF-IDF + embeddings
      ├── antec-storage/             # SQLite pool, migrations, repos
      ├── antec-security/            # Injection detection, vault, audit
      ├── antec-skills/              # Skill loader, 5 runtimes
      ├── antec-scheduler/           # Cron, NL scheduling, heartbeat
      ├── antec-sandbox/             # WASM + OS sandbox, policy engine
      ├── antec-mcp/                 # MCP client (stdio/SSE/HTTP)
      ├── antec-extensions/          # Integration templates, credentials
      ├── antec-i18n/                # Translation macro, locale files
      ├── antec-console/             # Web Console SPA (embedded)
      └── antec-hands/               # Operational capabilities
```

Dependency DAG: `prd/01-ARCHITECTURE.md` §1.2. **No circular dependencies.**

---

## Rust Coding Standards

Full standard: `.claude/skills/rust/SKILL.md` (179 rules, 14 categories). Key rules:

### Error Handling
- Library crates: `thiserror` for typed error enums per crate
- Application layer: `anyhow` for ergonomic propagation
- **Never** `.unwrap()` or `.expect()` in production code — use `?`
- Chain errors with context: `.map_err(|e| Error::Storage(e))?`

### Ownership & Borrowing
- Borrow (`&T`) over clone. Clone only when ownership transfer is required
- `&str` over `String` in function parameters
- `Arc<T>` for shared ownership in async — never `Rc<T>` in async code
- `tokio::sync::Mutex` (not `std::sync::Mutex`) when holding across `.await`

### Async Patterns
- Tokio-only runtime. Never `block_on` inside async
- Never hold `std::sync::Mutex` across `.await`
- CPU-heavy work: `tokio::task::spawn_blocking`
- Channels: `mpsc` (fan-in), `broadcast` (fan-out), `watch` (config), `oneshot` (request-reply)
- Parallel tasks: `JoinSet` with bounded concurrency

### API Design
- Newtype wrappers for IDs: `SessionId(Uuid)`, not bare `Uuid`
- Builder pattern for complex structs (3+ optional fields)
- Accept `impl Into<String>`, return concrete types
- Traits for plugin boundaries: `LlmProvider`, `Channel`, `MemoryBackend`, `Tool`

### Naming
- Types: `PascalCase`. Functions: `snake_case`. Constants: `SCREAMING_SNAKE_CASE`
- Constructors: `new()`, `with_*()`. Conversions: `as_*()` (cheap), `to_*()` (expensive), `into_*()` (ownership)
- Booleans: `is_*()`, `has_*()`, `can_*()`

### Memory & Performance
- `Vec::with_capacity()` when size is known
- `&[u8]` and `Cow<'_, str>` in parsing hot paths
- `SmallVec` for small collections (< 8 elements)
- `String::with_capacity` + `push_str`, not repeated `format!`

---

## Architecture Constraints

### Dependency Rules
- Crates form a DAG. No circular dependencies
- Leaf crates (`antec-storage`, `antec-i18n`) depend only on external crates
- Cross-crate communication through traits

### Config Pattern
Every new config field needs: struct field with `#[serde(default)]`, `Default` impl entry, serde derives, docs in `prd/14-CONFIGURATION.md`.

### Route Registration
New API routes need: handler in route module, router registration in `server.rs`, request/response types with serde, integration test.

### Tool Registration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rkinas/antec](https://github.com/rkinas/antec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
