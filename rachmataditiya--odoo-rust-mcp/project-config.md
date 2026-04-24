---
trigger: always_on
description: A **dual-stack Model Context Protocol (MCP) server** for Odoo integration:
---

# Odoo Rust MCP Server - AI Coding Agent Instructions

## 🎯 Project Overview

A **dual-stack Model Context Protocol (MCP) server** for Odoo integration:
- **Backend**: Rust (`rust-mcp/`) - MCP protocol implementation, Odoo client abstraction, tools execution
- **Frontend**: React TypeScript (`config-ui/`) - Web-based configuration editor for servers, tools, prompts, and multi-instance Odoo connections
- Supports **Odoo 19+** (JSON-2 External API with API keys) and **Odoo <19** (JSON-RPC with username/password)

## 🏗️ Architecture Patterns

### Dual Client Pattern (`src/odoo/`)
The codebase maintains **abstraction over two Odoo authentication methods**:
- **Modern**: `OdooClient` (Odoo 19+) - JSON-2 API with API key auth
- **Legacy**: `LegacyClient` (Odoo <19) - JSON-RPC with username/password auth
- **Unified**: `OdooClient` trait abstracts both implementations

**Key file**: `src/odoo/unified_client.rs` - All tool operations go through this abstraction.

### MCP Handler Pattern (`src/mcp/`)
Core request handler implements `ServerHandler` trait with these methods:
- `tools/list` - Returns tools from registry (tools.json)
- `tools/call` - Routes to `execute_op()` in `mcp/tools.rs`
- `prompts/list`, `prompts/get` - Returns prompts from registry
- `resources/list`, `resources/read` - Returns Odoo instance metadata via `odoo://` URIs

**Tool execution flow**: `tools/call` → `execute_op()` → `op_search`, `op_read`, `op_create`, etc. → Odoo client

### Config Manager Architecture (`src/config_manager/`)
**Hot-reload capable** configuration system:
- **Manager** (`manager.rs`) - CRUD operations for JSON configs (instances, tools, prompts, server)
- **Watcher** (`watcher.rs`) - File system monitoring for external changes
- **Server** (`server.rs`) - HTTP REST API and Axum web server on port 3008
- **React UI** (`config-ui/`) - Consumes REST API, syncs to manager

**Critical workflow**: File change → Watcher detects → Registry reloads → MCP handler reflects changes

### Registry Pattern (`src/mcp/registry.rs`)
Centralized configuration store:
- Loads `tools.json`, `prompts.json`, `server.json` from `~/.config/odoo-rust-mcp/`
- Caches parsed tool definitions with guard evaluation (`requiresEnv`, `requiresEnvTrue`)
- Guards gate access to dangerous tools (cleanup tools require `ODOO_ENABLE_CLEANUP_TOOLS=true`)

## 🔄 Key Workflows

### Adding a New Tool
1. Define tool structure in `src/mcp/registry.rs` (e.g., `OpSpec` with `op_type` and `map`)
2. Implement handler function: `async fn op_my_tool(pool: &OdooClientPool, op: &OpSpec, args: Value) -> Result<Value, OdooError>`
3. Add `op_type` case in `execute_op()` router
4. Add JSON definition to `config/tools.json`
5. Test with unit tests in `src/mcp/tools.rs` module tests

### Testing
- **Rust**: `cargo test --lib` (193 lib tests), `cargo test --all-targets` (includes integration tests in `tests/`)
- **TypeScript**: `npm test` (205 tests), `npm run test:coverage` (Istanbul provider)
- **CI**: GitHub Actions (parallel: build-ui, check, fmt, clippy, test, test-multiplatform, coverage)
- **Important**: Don't use `2>&1` in commands - stdout/stderr already go to terminal

### Code Quality
- **Format**: `cargo fmt --all` before commit
- **Lint**: `cargo clippy --all-targets --all-features -- -D warnings` (must pass)
- Both checked in CI (`fmt` and `clippy` jobs)

### Build Outputs
- **Rust Binary**: `target/debug/rust-mcp` or `target/release/rust-mcp`
- **UI Static**: Built to `rust-mcp/static/dist/` (embedded in binary via `include_dir!`)
- **Coverage**: Rust via `cargo-tarpaulin`, TS via `vitest` with `@vitest/coverage-istanbul`

## 📋 Project Structure

```
rust-mcp/                    # Rust server (Cargo workspace)
  src/
    main.rs                  # CLI entry point (stdio, HTTP, WebSocket transports)
    lib.rs                   # Library root (pub mod cleanup, config_manager, mcp, odoo)
    mcp/
      mod.rs                 # McpOdooHandler (ServerHandler implementation)
      tools.rs               # Tool execution logic (1400+ lines, helper functions)
      registry.rs            # Config loading, tool/prompt definitions
      cursor_stdio.rs        # Stdio transport for Cursor/Claude Desktop
      http.rs                # Axum server with MCP-over-HTTP transport
      cache.rs               # Metadata caching with TTL
      prompts.rs             # Hardcoded prompts for domain-specific guidance
      resources.rs           # odoo:// URI scheme parsing
      runtime.rs             # ServerCompat wrapper for MCP compatibility
    odoo/
      unified_client.rs      # OdooClient trait (abstraction over modern + legacy)
      client.rs              # Modern JSON-2 API client (Odoo 19+)
      legacy_client.rs       # Legacy JSON-RPC client (Odoo <19)
      config.rs              # Instance config parsing and auth mode detection
      types.rs               # OdooError, serialization types
    config_manager/
      manager.rs             # Config CRUD (load/save instances, tools, prompts)
      server.rs              # Axum HTTP server (REST API, UI serving)
      watcher.rs             # File watcher for hot reload
      mod.rs                 # Exports
    cleanup/
      database.rs            # Database cleanup tool

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rachmataditiya/odoo-rust-mcp](https://github.com/rachmataditiya/odoo-rust-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
