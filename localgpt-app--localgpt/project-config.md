---
trigger: always_on
description: This file provides guidance to GitHub Copilot when working with code in this repository.
---

# GitHub Copilot Instructions for LocalGPT

This file provides guidance to GitHub Copilot when working with code in this repository.

## Project Overview

LocalGPT is a local-only AI assistant built in Rust with persistent markdown-based memory and optional autonomous operation via heartbeat. It runs entirely on your machine with no cloud dependency required.

## Quick Reference

### Build & Test Commands

```bash
# Build
cargo build                     # Debug build (default-members = cli)
cargo build --release           # Release build
cargo build --workspace         # Build all crates

# Test
cargo test --workspace          # All tests
cargo test -p localgpt-core     # Single crate
cargo test -- --nocapture       # Show stdout

# Lint (REQUIRED before commits)
cargo clippy --workspace
cargo fmt --check

# Run
cargo run -- chat               # Interactive chat
cargo run -- ask "question"     # Single question
cargo run -- daemon start       # HTTP server + Telegram bot + heartbeat
```

### Cross-platform Validation

```bash
# Mobile cross-compile checks (required for core changes)
cargo check -p localgpt-mobile-ffi --target aarch64-apple-ios
cargo check -p localgpt-mobile-ffi --target aarch64-apple-ios-sim
```

## Architecture

### Workspace Structure (7 crates)

```
crates/
├── core/        # localgpt-core — shared library (agent, memory, config, security)
├── cli/         # localgpt — binary with clap CLI, desktop GUI, dangerous tools
├── server/      # localgpt-server — HTTP/WS API, Telegram bot, optional WASM web UI
├── sandbox/     # localgpt-sandbox — Landlock/Seatbelt process sandboxing
├── mobile-ffi/  # localgpt-mobile-ffi — UniFFI bindings for iOS/Android
├── gen/         # localgpt-gen — Bevy 3D scene generation binary
└── bridge/      # localgpt-bridge — secure IPC protocol for bridge daemons

bridges/         # Standalone bridge binaries (Telegram, Discord, WhatsApp)
apps/            # Native mobile app projects (iOS, Android)
```

### Dependency Rules

**CRITICAL**: `localgpt-core` must have zero platform-specific dependencies. It must compile cleanly for `aarch64-apple-ios` and `aarch64-linux-android`. No clap, eframe, axum, teloxide, landlock, nix, etc.

```
localgpt ──→ localgpt-core
             ──→ localgpt-server ──→ localgpt-core
             ──→ localgpt-sandbox ──→ localgpt-core

localgpt-mobile-ffi ──→ localgpt-core (default-features = false, embeddings-openai)
localgpt-gen        ──→ localgpt-core
```

### Feature Flags (`localgpt-core`)

| Feature | Default | Purpose |
|---------|---------|---------|
| `embeddings-local` | yes | fastembed/ONNX local embeddings |
| `embeddings-openai` | no | OpenAI API embeddings (mobile uses this) |
| `embeddings-gguf` | no | llama.cpp GGUF embeddings |
| `embeddings-none` | no | FTS5 keyword search only |
| `claude-cli` | yes | ClaudeCliProvider (subprocess-based, excluded on mobile) |

Mobile crate uses `default-features = false, features = ["embeddings-openai"]` to exclude native dependencies.

## Key Design Patterns

### Tool Safety Split
- `Agent::new()` creates safe tools only (memory_search, memory_get, web_fetch, web_search)
- CLI injects dangerous tools (bash, read_file, write_file, edit_file) via `agent.extend_tools(create_cli_tools())`
- Server agents intentionally only get safe tools

### Thread Safety
- Agent is NOT `Send+Sync` due to SQLite
- Use `AgentHandle` (`Arc<tokio::sync::Mutex<Agent>>`) for mobile/server
- HTTP handler uses `spawn_blocking`

### Bevy Main Thread
- Bevy must own the main thread (macOS windowing/GPU requirement)
- Gen mode spawns tokio on a background thread

### Session Management
- When approaching context limits, compaction triggers a memory flush first
- LLM saves important context to MEMORY.md before messages are truncated
- New sessions automatically load `MEMORY.md`, recent daily logs, `HEARTBEAT.md`

### Path Expansion
- All tools use `shellexpand::tilde()` for `~` in paths

### Provider Routing
Model prefix determines LLM provider:
- `claude-cli/*` → Claude CLI
- `gpt-*`/`openai/*` → OpenAI
- `claude-*`/`anthropic/*` → Anthropic API
- `glm-*`/`glm/*` → GLM (Z.AI)
- `ollama/*` → Ollama

## Coding Standards

### General Guidelines
- Follow Rust idioms and best practices
- Use `anyhow::Result` for application errors, `thiserror` for library errors
- Prefer `tracing` macros over `println!` for logging
- Use structured logging with context fields
- All public APIs should have documentation comments

### Error Handling
- Propagate errors with `?` operator where appropriate
- Provide context with `.context()` or `.with_context()`
- Use specific error types for recoverable errors
- Fatal errors should be logged with `tracing::error!` before panicking

### Testing
- Unit tests go in `#[cfg(test)] mod tests` at end of file
- Integration tests go in `tests/` directory
- Use `#[tokio::test]` for async tests
- Mock external dependencies (filesystem, network, LLM APIs)

### Commit Conventions
- Use conventional commit format: `type(scope): description`
- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
- Keep commits atomic and focused
- Reference issue numbers in commit messages

## Security Considerations

### Sandbox

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [localgpt-app/localgpt](https://github.com/localgpt-app/localgpt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
