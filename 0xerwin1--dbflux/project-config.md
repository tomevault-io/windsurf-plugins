---
trigger: always_on
description: Guidelines for AI agents working in this Rust/GPUI codebase.
---

# AGENTS.md — DBFlux

Guidelines for AI agents working in this Rust/GPUI codebase.

## Project Overview

DBFlux is a keyboard-first database client built with Rust and GPUI (Zed's UI framework).

**Workspace structure:**

```
crates/
├── dbflux/                    # Binary shell: main entry point, CLI, IPC server
├── dbflux_ui/                 # GPUI UI layer: views, documents, overlays, components, keymap
├── dbflux_app/                # Runtime/domain: AppState, managers, hooks, auth providers
├── dbflux_core/               # Traits, types, errors, driver capabilities (stable API)
├── dbflux_ipc/                # IPC envelopes, framing, and driver RPC protocol
├── dbflux_driver_ipc/         # External driver proxy over local IPC
├── dbflux_driver_host/        # RPC host process for out-of-process drivers
├── dbflux_driver_postgres/    # PostgreSQL driver
├── dbflux_driver_sqlite/      # SQLite driver
├── dbflux_driver_mysql/       # MySQL/MariaDB driver
├── dbflux_driver_mongodb/     # MongoDB driver
├── dbflux_driver_redis/       # Redis driver
├── dbflux_driver_dynamodb/    # DynamoDB driver
├── dbflux_aws/                # AWS auth providers + AWS value providers
├── dbflux_ssm/                # AWS SSM managed tunnel factory
├── dbflux_lua/                # Embedded Lua runtime for in-process hooks
├── dbflux_tunnel_core/        # Shared RAII tunnel infrastructure (proxy + SSH)
├── dbflux_proxy/              # SOCKS5/HTTP CONNECT proxy tunnel
├── dbflux_ssh/                # SSH tunnel support
├── dbflux_export/             # Export (CSV, JSON, Text, Binary)
├── dbflux_test_support/       # Docker containers and fixtures for integration tests
├── dbflux_mcp/                # MCP runtime, governance service, and tool catalog
├── dbflux_mcp_server/         # Standalone MCP server binary for AI clients
├── dbflux_policy/             # Policy engine, roles, trusted clients, classification
├── dbflux_approval/           # Approval service and pending execution store
├── dbflux_audit/              # Audit logging with SQLite backend
└── dbflux_storage/            # Unified storage: SQLite database, migrations, repositories
```

## Build & Run Commands

```bash
cargo check --workspace              # Fast type checking
cargo build -p dbflux --features sqlite,postgres,mysql,mongodb,redis,dynamodb,aws  # Debug build
cargo build -p dbflux --features sqlite,postgres,mysql,mongodb,redis,dynamodb,aws --release  # Release build
cargo run -p dbflux --features sqlite,postgres,mysql,mongodb,redis,dynamodb,aws    # Run app

# MCP server (AI integration) - included by default
cargo build -p dbflux  # MCP included in default features
./target/debug/dbflux mcp --client-id test-client

# Build without MCP support (smaller binary, no AI integration)
cargo build -p dbflux --no-default-features --features sqlite,postgres,mysql,mongodb,redis,dynamodb,lua,aws

cargo fmt --all                      # Format
cargo clippy --workspace -- -D warnings  # Lint
cargo test --workspace               # All tests
cargo test --workspace test_name     # Single test
cargo test -p dbflux_core            # Tests in specific crate
cargo test -p dbflux_driver_dynamodb --test live_integration -- --ignored  # Docker-backed live tests

# Nix
nix develop                          # Enter dev shell
nix build                            # Build package
nix run                              # Run directly
```

## Rust Guidelines

### General Principles

- Prioritize correctness and clarity over speed
- Do not write comments that summarize code; only explain non-obvious "why"
- Prefer implementing in existing files unless it's a new logical component
- Avoid creating many small files
- Avoid creative additions unless explicitly requested
- Use full words for variable names (no abbreviations like "q" for "queue")

### Error Handling

- Avoid `unwrap()` and functions that panic; use `?` to propagate errors
- Be careful with indexing operations that may panic on out-of-bounds
- Never silently discard errors with `let _ =` on fallible operations:
  - Propagate with `?` when the caller should handle them
  - Use `.log_err()` when ignoring but wanting visibility
  - Use `match` or `if let Err(...)` for custom logic
- Ensure async errors propagate to UI so users get meaningful feedback

### File Organization

- Use `mod.rs` for module directories (e.g., `views/mod.rs`, not a sibling `views.rs`)
- When creating crates, specify library root in `Cargo.toml` with `[lib] path = "..."`

### Async Patterns

Use variable shadowing to scope clones in async contexts:

```rust
executor.spawn({
    let task_ran = task_ran.clone();
    async move {
        *task_ran.borrow_mut() = true;
    }
});
```

### Performance Patterns

**Pre-compute expensive operations**: Move string formatting and allocation into constructors rather than during rendering:

```rust
// Good: Format once during construction
CellValue::Text { display: format!("{}", value), ... }

// Bad: Format on every render
fn render(&self) { format!("{}", self.value) }
```

**Lazy loading for large datasets**: Drivers should return shallow metadata initially and fetch details on-demand:

```rust
fn get_tables(&self) -> Vec<TableInfo> // Names only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xErwin1/dbflux](https://github.com/0xErwin1/dbflux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
