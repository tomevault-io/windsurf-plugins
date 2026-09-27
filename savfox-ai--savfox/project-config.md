---
trigger: always_on
description: This file provides guidance to Codex and other coding agents working in this repository.
---

# AGENTS.md

This file provides guidance to Codex and other coding agents working in this repository.

## Scope

- This is the root guide for the whole workspace.
- Directory-specific `AGENTS.md` files take precedence for their subtree.
- Today the main local override is `crates/tui/src/bottom_pane/AGENTS.md`, which adds doc-sync requirements for bottom-pane state machines.

## Source Of Truth

- Prefer `Cargo.toml`, `Justfile`, crate `README.md` files, and current source code over older prose docs when they disagree.
- Some docs still reference older names such as `codex-api` or `OpenClaw`; do not copy those names into new code or new documentation unless you are intentionally documenting compatibility behavior.

## Build And Test Commands

### Workspace-wide

```bash
cargo build --workspace
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all
cargo fmt --all -- --check
```

### Common targeted test commands

```bash
cargo test -p savfox-core
cargo test -p savfox-tui
cargo test -p savfox-exec
cargo test -p savfox-app-server
cargo test -p savfox-app-server-protocol
cargo test -p savfox-gateway-server
cargo test -p savfox-api-client
cargo test -p savfox-mcp-server
```

### CLI install

```bash
cargo install --path crates/savfox-cli
```

### `just` shortcuts

```bash
just check
just lint
just test
just fmt
just gateway
just gateway-release
just gateway-skip-web
just gateway-backend
just gateway-frontend
just web-build
just web-build-release
just web-serve
```

Notes:

- `just gateway` and `just gateway-release` run `scripts/build-web.ps1` before starting the gateway.
- `just gateway-backend` runs only the gateway server on the fixed dev port `18881` and is meant to pair with `just gateway-frontend`.
- `just gateway-frontend` runs `dx serve --web` for the Dioxus app; its dev server proxies `/api`, `/health`, and `/ws` back to the local gateway backend on port `18881`.
- `just gateway-skip-web` and `just web-serve` remain available as compatibility aliases for the split backend/frontend dev loop.

## Workspace Architecture

Savfox is a large Rust workspace centered on `savfox-core`, with multiple product surfaces layered on top of it.

```text
savfox-cli / savfox-exec / savfox-tui
                |
            savfox-core
      _________|____________
     |         |            |
 protocol   api-client   app-server / mcp-server
     |                        |
 gateway-server -------- gateway-dioxus
     |
  channels / cron / web / ws-rpc
```

### Main crates

- `crates/savfox-cli`: the `savfox` binary. It dispatches to the interactive TUI when no subcommand is given and exposes commands like `exec`, `review`, `login`, `gateway`, `app-server`, `mcp`, `sessions`, `skills`, `daemon`, `cron`, `doctor`, and `sandbox`.
- `crates/exec`: powers `savfox exec` and `savfox review`. In normal mode stdout is reserved for final user-facing output; in `--json` mode stdout must remain valid JSONL.
- `crates/tui`: the interactive Ratatui/Crossterm UI. This crate has extensive snapshot coverage and its style guide lives in `crates/tui/styles.md`.
- `crates/core`: the main agent engine. It owns config loading, auth, session management, tool orchestration, sandboxing, MCP integration, skills, memory, review flow, rollouts, and shared session behavior used by all frontends.
- `crates/protocol`: shared protocol and event types used across `core`, `exec`, `tui`, and app/gateway surfaces. Keep business logic light here.
- `crates/api-client` and `crates/http-client`: typed wire/API layer for model providers and streaming responses. `core` consumes these rather than embedding HTTP protocol details directly.
- `crates/app-server` and `crates/app-server-protocol`: stdio JSON-RPC server and protocol types for IDE/editor integrations. Current exported protocol types are generated from `src/protocol/common.rs`, `src/protocol/session_history.rs`, and `src/protocol/v1.rs`.
- `crates/mcp-server` and `crates/rmcp-client`: Savfox as an MCP server plus support for external MCP servers/tools.
- `crates/gateway-server`: the remote HTTP/WebSocket gateway built on Salvo. It owns ws-rpc dispatch, auth, cron jobs, channel routing, webchat, browser/voice helpers, session bridging, and gateway-side state.
- `crates/gateway-dioxus` and `crates/gateway-shared`: the web frontend and its shared serde types. The built frontend is copied into `crates/gateway-server/static`.
- `crates/channels`: channel implementations for external platforms such as Discord, Telegram, Slack, Matrix, Mattermost, Feishu, IRC, Webhook, and others used by the gateway.
- `crates/memory`, `crates/skill-registry`, `crates/browser-automation`, `crates/network-proxy`, `crates/linux-sandbox`, and `crates/windows-sandbox`: supporting subsystems for long-term memory, skills, browser tooling, network policy, and platform sandboxing.
- `apps/` and `apps/shared/SavfoxKit`: Apple and Android client surfaces that talk to Savfox services rather than reimplementing the core agent stack.

## Project Conventions

- Rust edition is `2024`; workspace `rust-version` is `1.98`.
- Crate names are consistently prefixed with `savfox-`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [savfox-ai/savfox](https://github.com/savfox-ai/savfox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
