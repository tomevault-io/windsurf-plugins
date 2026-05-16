---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

mcproc is a Rust-based daemon that fulfills Model Context Protocol (MCP) tool calls from LLMs and manages multiple development processes. The project provides a robust process management system with comprehensive logging, validation, and monitoring capabilities.

## Key Architecture Components

### High-Level Architecture

```
Claude / Other LLMs
      │  JSON-RPC 2.0 (MCP) via stdio
      ▼
┌───────────────────────────────┐
│   mcproc (CLI + MCP server)   │
│───────────────────────────────│
│  • MCP Tool Handlers          │
│  • gRPC Client                │
└───────────────────────────────┘
      │ gRPC (Unix Domain Socket)
      ▼
┌───────────────────────────────┐
│   mcproc daemon (--daemon)    │
│───────────────────────────────│
│  • Process Manager            │
│  • Log Hub (direct file I/O)  │
│  • gRPC API (tonic)           │
└───────────────────────────────┘
      │ stdout/stderr capture
      ▼
┌────────────────────────────────┐
│  Child Processes               │
│  (npm run dev, python app.py,  │
│   cargo run, etc.)             │
└────────────────────────────────┘
```

### Core Components
- **mcproc daemon**: The main daemon process (runs as `mcproc --daemon`)
  - Process Manager for spawning/managing child processes
  - Log Hub with file-based logging (temporary, deleted on system reboot)
  - API Layer using tonic (gRPC via Unix Domain Socket)
  - Port detection and monitoring
  - Process state tracking (Starting, Running, Stopping, Stopped, Failed)

- **mcproc**: CLI tool for interacting with the daemon
  - Communicates via gRPC Unix socket
  - Supports commands: start, stop, restart, ps, logs, grep, clean, mcp serve
  - Project-based process organization
  - Auto-starts daemon if not running

- **mcp-rs**: Reusable MCP library
  - ServerBuilder for creating MCP servers
  - Transport implementations (stdio implemented, SSE/HTTP planned)
  - Tool registry and handler interfaces

### MCP Tools
The daemon exposes these tools to LLMs via `mcproc mcp serve`:
- `start_process`: Start and manage development processes
- `stop_process`: Terminate processes
- `restart_process`: Restart running processes
- `list_processes`: List all managed processes
- `get_logs`: Fetch or stream process logs
- `get_process_status`: Get detailed process information
- `search_process_logs`: Search through logs with regex patterns

## Prerequisites

- Rust toolchain (rustc, cargo)
- protobuf compiler: **REQUIRED** - Install with:
  - macOS: `brew install protobuf`
  - Linux: `apt-get install protobuf-compiler`
  - Without this, the build will fail with "Could not find `protoc`" error

## Development Commands

```bash
# Build - IMPORTANT: Always use --all-targets to check binaries
cargo build --all-targets  # Check all targets including binaries
cargo build --release --all-targets

# Test
cargo test
cargo test -- --nocapture  # Show println! output

# Lint - Include all targets
cargo clippy --all-targets -- -D warnings

# Format
cargo fmt
cargo fmt -- --check  # Check without modifying

# Check before install (detect binary compilation errors early)
cargo check --bin mcproc  # Check binary compilation
cargo install --path mcproc --dry-run  # Dry run to detect install errors

# Run
cargo run --bin mcproc -- --daemon  # Run daemon (hidden option)
cargo run --bin mcproc -- daemon start  # Run daemon (recommended)
cargo run --bin mcproc -- <command>  # Run CLI
```

### Pre-commit Checklist

**🚨 MANDATORY - NEVER commit without completing ALL steps below 🚨**

Run these checks in order. If ANY step fails, STOP and fix before proceeding:

```bash
# 1. Format check (MUST pass)
cargo fmt -- --check

# 2. Clippy (linting) - Include all targets (MUST pass with zero warnings)
cargo clippy --all-targets -- -D warnings

# 3. Build check - Include binaries (MUST compile successfully)
cargo build --all-targets

# 4. Tests (MUST pass)
cargo test

# 5. Binary check (ensure mcproc can be installed)
cargo check --bin mcproc

# 6. Security audit (review any issues)
cargo audit
```

**Verification command (all-in-one check):**
```bash
cargo fmt -- --check && \
cargo clippy --all-targets -- -D warnings && \
cargo build --all-targets && \
cargo test
```

**Failure Protocol:**
- If format check fails: Run `cargo fmt` then re-check
- If clippy fails: Fix ALL warnings before proceeding (no #[allow] without permission)
- If build fails: Fix compilation errors
- If tests fail: Fix failing tests

**NO EXCEPTIONS**: Quality gates are mandatory for every commit, regardless of change size.

## Project Structure

The project uses a Cargo workspace with the following crates:
- `mcp-rs/` - Reusable MCP library for creating MCP servers
- `mcproc/` - Main crate containing both daemon and CLI
  - `src/daemon/` - Daemon implementation (process management, gRPC server)
  - `src/cli/` - CLI commands and MCP tools
  - `src/common/` - Shared utilities (config, validation, status)
  - `src/client/` - gRPC client for daemon communication
- `proto/` - Protocol buffer definitions for gRPC services

## Implementation Specifications

### Logging

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [neptaco/mcproc](https://github.com/neptaco/mcproc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
