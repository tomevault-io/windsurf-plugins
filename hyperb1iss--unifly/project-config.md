---
trigger: always_on
description: handles this.
---

# AGENTS.md

Instructions for AI agents working on the **unifly** codebase. This file is the
single source of truth for repository conventions. `CLAUDE.md` is a symlink to
this file. Cursor, Codex, Aider, Cline, and Claude Code all read from here.

For end-user documentation see `README.md`. For contributor workflow see
`CONTRIBUTING.md`. For how agents should _use_ the unifly CLI at runtime see
`skills/unifly/SKILL.md` (a separate audience from this file).

---

## What This Repository Is

unifly is a Rust CLI and TUI for managing Ubiquiti UniFi network
infrastructure. A single `unifly` binary ships three user-facing surfaces:

- **CLI commands** (`unifly devices list`, etc.): 27 top-level commands
  covering devices, clients, networks, WiFi, firewall, NAT, DNS, ACL,
  traffic-lists, hotspot, events, stats, DPI, VPN, Wi-Fi observability
  (neighbors, channels, roams, experience), cloud fleet queries, and a raw
  `api` escape hatch.
- **TUI dashboard** (`unifly tui`) -- 10-screen Ratatui interface for real-time
  monitoring and interactive management.
- **Agent skill** at `skills/unifly/SKILL.md`: bundled documentation that
  teaches AI agents to drive the CLI.

The binary is powered by the `unifly-api` library crate, which is also
published independently on crates.io for Rust developers building custom
integrations.

The moat is **triple-path coverage**: unifly speaks the modern Integration
API (REST, API key), the Session API (cookie + CSRF), and Site Manager cloud
fleet/connector APIs. Most competing tools are one path or two; unifly can
mix local and cloud workflows in one binary.

---

## Quick Command Reference

The project uses **just** for task orchestration. All recipes live in the
`justfile` at the repo root.

```bash
# The canonical "before commit" gate
just check              # fmt-check + clippy + test

# Individual gates
just fmt-check          # nightly rustfmt, read-only
just clippy             # cargo clippy --workspace --all-targets
just test               # cargo test --workspace

# Fix-it automation
just fix                # clippy --fix + cargo fmt --all
just fmt                # cargo fmt --all

# Running
just cli <args>         # cargo run -p unifly -- <args>
just tui <args>         # cargo run -p unifly -- tui <args>

# Targeted tests
just test-crate unifly-api
just test-verbose       # cargo test --workspace -- --nocapture
just snap-review        # cargo insta review for snapshot tests

# Build
just build              # debug build (workspace)
just build-release      # release build (workspace)

# Install
just install            # cargo install --path crates/unifly (CLI + TUI)
just install-cli        # CLI without TUI dependencies

# Docs and cleanup
just doc                # cargo doc --workspace --open
just clean              # cargo clean
```

**Always run `just check` before committing.** It is the same gate CI runs.

---

## Workspace Layout

This is a **2-crate Cargo workspace** (not 5, despite older notes):

```
crates/
  unifly-api/         # Library: HTTP/WS transport, Controller, DataStore, model
  unifly/             # Single binary: CLI commands + TUI dashboard (feature-gated)
```

Dependency chain: `unifly` depends on `unifly-api`.

**Edition 2024. MSRV 1.94. Resolver 3. Workspace version is pinned in
`[workspace.package]` at `Cargo.toml`.** All member crates inherit the
version via `version.workspace = true`.

Published to crates.io: `unifly-api`, `unifly`. Private/internal artifacts
live under `aur/`, `homebrew-tap` (separate repo), and `skills/unifly/` (for
ClawHub and Claude Code plugin).

---

## Architecture Essentials

### Dual-API Transport

`crates/unifly-api/src/` splits transport into two clients:

- **`integration/`**: `IntegrationClient`, `X-API-KEY` auth, modern REST
  endpoints at `/proxy/network/integration/v1/`. Returns clean JSON with
  UUIDs. Covers configuration CRUD (networks, wifi, firewall, nat, dns,
  acl, hotspot, traffic-lists, wans).
- **`session/`**: `SessionClient`, session cookie + CSRF for session auth,
  plus `X-API-KEY` on UniFi OS session HTTP endpoints. Uses envelope-wrapped
  responses at `/proxy/network/api/` and `/proxy/network/v2/api/`. Covers
  events, stats, Wi-Fi/client observability, device commands, admin,
  backups, DPI control. Session WebSocket still requires a session cookie.

Both clients share a common `TransportConfig` and `TlsMode`. TLS modes:
`SystemDefaults`, `AcceptInvalid`, `CustomPem`. Credentials are wrapped in
`secrecy::SecretString` throughout. **Never log or display them.**

### Controller Facade

`crates/unifly-api/src/controller/` wraps everything behind a single
`Controller` type:

```rust
pub struct Controller(Arc<ControllerInner>);
```

The `Arc` makes `Controller` cheaply cloneable across async tasks. Key
operations live in submodules:

- `lifecycle.rs`: connect, disconnect, reconnect
- `runtime.rs`: background refresh loop, command processor
- `query.rs` / `query/`: read-side operations
- `commands/`: write-side operations (`execute(CoreCommand)`)
- `payloads.rs` / `payloads/`: request body construction
- `refresh.rs`: periodic data sync
- `subscriptions.rs`: WebSocket event fan-out
- `session_queries.rs`: Session-specific read paths, including `raw_get`/
  `raw_post` used by the `unifly api` command

### Reactive DataStore


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyperb1iss/unifly](https://github.com/hyperb1iss/unifly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
