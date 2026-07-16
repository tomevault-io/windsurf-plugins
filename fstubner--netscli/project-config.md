---
trigger: always_on
description: Instructions for AI coding agents working on this codebase.
---

# AGENTS.md - NetsCLI

Instructions for AI coding agents working on this codebase.

## Project Overview

NetsCLI is a cross-platform network scanning and diagnostics toolkit in Rust. Four interfaces share one core library: interactive TUI, CLI subcommands, desktop GUI (Tauri 2 + React 19), and an MCP server for AI agent integration.

## Repository Layout

```
netscli/
  crates/
    netscli-core/       # Core networking library (all scan/discovery logic)
    netscli-mcp/        # MCP server (JSON-RPC 2.0 over stdio)
  apps/
    netscli-cli/        # CLI binary + TUI (ratatui/crossterm)
    netscli-gui/        # Desktop GUI (Tauri 2 + React 19 + TypeScript)
      src/              #   React frontend
      src-tauri/        #   Tauri Rust backend
  scripts/              # OUI generator, install scripts (excluded from workspace)
  data/                 # Static datasets (oui.min.json.gz)
```

**Dependency flow**: the `netscli` package (at `apps/netscli-cli/`) and `netscli-gui/src-tauri` depend on `netscli-core`. `netscli-mcp` wraps `netscli-core`. The `netscli` package also depends on `netscli-mcp`.

**Crate name vs source directory**: the CLI source lives at `apps/netscli-cli/` to match the other workspace members (`-core`, `-mcp`, `-gui`), but the Cargo package is just `netscli` — so end users run `cargo install netscli` (matching the produced binary). Internally we use `-p netscli`.

For ownership boundaries and compatibility rules, see `docs/ARCHITECTURE.md`.

## Build, Test, and Development Commands

```bash
# CLI/TUI (binary name: netscli)
cargo build -p netscli
cargo build -p netscli --features pcap   # with packet capture
cargo build --release -p netscli          # optimized build

# GUI
cd apps/netscli-gui && npm install && npm run tauri dev
cd apps/netscli-gui && npm run tauri build    # build installer

# Tests
cargo test --all                              # all workspace tests
cargo test -p netscli-core                    # targeted test run
./scripts/test-pcap.ps1                       # Windows PCAP tests with Npcap SDK env
cd apps/netscli-gui && npm run test:unit      # GUI helper tests
cd apps/netscli-gui && npm run build          # GUI typecheck + Vite build
cd apps/netscli-gui && npm run test:tauri-render  # Tauri render automation

# Linting & formatting
cargo fmt                                     # apply rustfmt
cargo clippy --all-targets -- -D warnings     # lint with warnings as errors

# OUI database refresh
cd scripts && cargo run --bin generate-oui
```

## Toolchain

- Rust **1.96.0** pinned in `rust-toolchain.toml` (includes rustfmt + clippy)
- MSRV: 1.96
- Cargo resolver: v2
- GUI frontend: Node.js with npm, Vite, TypeScript

## Key Dependencies

| Crate/Lib | Role |
|-----------|------|
| `tokio` (full) | Async runtime, used everywhere |
| `serde` + `serde_json` | Serialization throughout |
| `serde_yaml_ng` | YAML output in CLI |
| `anyhow` | Error handling in apps |
| `thiserror` | Typed errors in library crates |
| `sqlx` (sqlite, async) | Database for host records and scan history |
| `pnet_packet/transport/datalink` | Raw networking |
| `hickory-resolver` | Async DNS resolution |
| `pcap` | Packet capture (optional feature) |
| `clap` 4.4 (derive) | CLI argument parsing |
| `ratatui` 0.29 + `crossterm` 0.27 | Terminal UI |
| `tauri` 2.0 | Desktop GUI framework |
| React 19 + Vite + TypeScript | GUI frontend |

## Architecture Rules

### Where to put new code

- **Network logic** (scanning, pinging, DNS, etc.): `crates/netscli-core/src/`
- **New network operations**: Add to the `ops.rs` facade and the matching `ops/` family module so all interfaces get it
- **CLI subcommands**: `apps/netscli-cli/src/args.rs` (clap) + `cli_dispatch/` (handler)
- **TUI commands**: `apps/netscli-cli/src/tui/events/`
- **TUI output formatting**: `apps/netscli-cli/src/tui_formatter/`
- **CLI text output**: `apps/netscli-cli/src/cli_formatter/`
- **MCP tool exposure**: `crates/netscli-mcp/src/server/`
- **GUI backend commands**: `apps/netscli-gui/src-tauri/src/commands/`
- **GUI frontend**: `apps/netscli-gui/src/`

Do not add GUI-only, TUI-only, CLI-only, or MCP-only network logic. Interface layers should call `netscli-core` or expose a missing core operation through `Ops`.

### Error handling pattern

- Library crates (`netscli-core`): use `thiserror` for typed error enums
- Application crates (`netscli-cli`, `netscli-gui`): use `anyhow::Result`
- MCP server: JSON-RPC error codes (-32600 to -32603)

### Async pattern

All network operations are async (tokio). Long-running operations accept progress callbacks for UI updates. Use `tokio::spawn` for concurrent work within operations.

### Platform-specific code

- Guard with `#[cfg(unix)]` / `#[cfg(windows)]`
- Windows: `ipconfig` crate for interfaces, `windows-sys` for WinSock, `tracert` command
- Unix: `pnet_datalink`/`pnet_sys` for raw sockets, manual ICMP TTL probes for traceroute

## Core Modules (netscli-core/src/)

| File | Purpose |
|------|---------|
| `lib.rs` | Public API re-exports |
| `ops.rs` | High-level operations facade (used by CLI, TUI, GUI, MCP) |
| `common.rs` | Default constants (ports, timeouts, concurrency) |
| `discover.rs` | Subnet host discovery (ping + DNS resolve) |
| `scan.rs` | TCP port scanning with concurrency |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fstubner/netscli](https://github.com/fstubner/netscli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
