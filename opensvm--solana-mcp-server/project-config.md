---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Solana MCP Server is a Model Context Protocol (MCP) implementation in Rust that provides comprehensive access to Solana blockchain data. It acts as a bridge between AI assistants (like Claude) and Solana RPC endpoints, enabling blockchain queries through natural language conversations.

## Directory Structure

```
.
├── benches/              # Criterion benchmarks (HTTP, RPC, WebSocket)
├── _docs/                # Source documentation (markdown)
├── docs/                 # Generated HTML docs + guides
├── k8s/                  # Kubernetes deployment configs
├── scripts/              # Deployment scripts (docker, k8s, lambda, etc.)
├── src/
│   ├── docs/             # Embedded documentation modules
│   ├── rpc/              # RPC method implementations by category
│   ├── server/           # ServerState and core server logic
│   ├── tools/            # MCP tool definitions and handlers
│   ├── x402/             # Payment protocol (feature-gated)
│   ├── cache.rs          # RPC response caching
│   ├── config.rs         # Configuration loading
│   ├── http_server.rs    # Axum HTTP server
│   ├── main.rs           # CLI entry point
│   ├── metrics.rs        # Prometheus metrics
│   ├── protocol.rs       # MCP protocol types
│   ├── transport.rs      # JSON-RPC transport
│   ├── validation.rs     # Input validation
│   └── websocket_server.rs
├── tests/                # Integration tests
├── Cargo.toml            # Rust dependencies
├── config.json           # Server configuration
└── netlify.toml          # Netlify deployment config
```

## Requirements

- Rust 1.75+
- MCP Protocol Version: `2025-06-18`

## Build Commands

```bash
cargo build                    # Debug build
cargo build --release          # Release build
cargo build --features x402    # With payment protocol

cargo test                     # Run all tests
cargo test <test_name>         # Run specific test
cargo test -- --nocapture      # Run with stdout/stderr output
cargo test --test <file_name>  # Run specific integration test

cargo bench                    # Run benchmarks
cargo clippy                   # Lint
cargo fmt                      # Format
cargo audit                    # Security vulnerability scan (install: cargo install cargo-audit)
```

## Running the Server

```bash
# Stdio mode (default, for Claude Desktop integration)
cargo run
# or
cargo run -- stdio

# Web service mode (HTTP API)
cargo run -- web --port 3000

# WebSocket mode (subscriptions)
cargo run -- websocket --port 8900
```

**Web API Endpoints:**
- `POST /api/mcp` - MCP JSON-RPC API
- `GET /health` - Health check
- `GET /metrics` - Prometheus metrics

**Important:** In stdio mode, all logging MUST go to stderr. Logging to stdout corrupts the JSON-RPC protocol. The server handles this automatically via `init_logging(_, true)` for stderr-only output.

## Architecture

### Core Modules (`src/`)

- **`main.rs`** - CLI entry point using clap. Handles three modes: stdio, web, websocket
- **`lib.rs`** - Public API exports
- **`server/mod.rs`** - `ServerState` struct that holds RPC clients, config, and cache. Central state management
- **`tools/mod.rs`** - MCP tool definitions and request handlers. Maps MCP tool calls to Solana RPC methods
- **`transport.rs`** - JSON-RPC transport layer for stdio communication
- **`protocol.rs`** - MCP protocol types (InitializeRequest, ToolDefinition, etc.)

### RPC Implementation (`src/rpc/`)

Organized by category:
- `accounts.rs` - getAccountInfo, getBalance, getProgramAccounts
- `blocks.rs` - getBlock, getBlockHeight, getBlockTime
- `tokens.rs` - getTokenAccountBalance, getTokenSupply
- `transactions.rs` - getTransaction, getSignaturesForAddress
- `system.rs` - getHealth, getVersion, getClusterNodes
- `missing_methods.rs` - Additional RPC methods

### Supporting Systems

- **`cache.rs`** - TTL-based LRU cache using DashMap for RPC response caching
- **`config.rs`** - Configuration loading (config.json or env vars). Supports multi-network SVM configs
- **`http_server.rs`** - Axum-based HTTP server for web mode
- **`websocket_server.rs`** - WebSocket server for real-time subscriptions
- **`metrics.rs`** - Prometheus metrics integration
- **`validation.rs`** - Input validation and sanitization utilities
- **`x402/`** - Optional payment protocol integration (feature-gated)

### Request Flow

1. Client sends JSON-RPC request via stdio/HTTP/WebSocket
2. `tools/mod.rs` routes to appropriate handler based on method name
3. Handler validates params via `validation.rs`
4. RPC call made via `ServerState.rpc_client` (or multi-network via `svm_clients`)
5. Response cached if enabled, then returned to client

### Multi-Network Support

The server can query multiple SVM-compatible networks simultaneously. Networks are configured in `config.json` under `svm_networks`. Each network has its own RPC client in `ServerState.svm_clients`.

## Adding New RPC Methods

1. **Create function in `src/rpc/<category>.rs`:**
```rust
pub async fn get_something(client: &RpcClient, param: &Pubkey) -> McpResult<Value> {
    let request_id = new_request_id();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openSVM/solana-mcp-server](https://github.com/openSVM/solana-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
