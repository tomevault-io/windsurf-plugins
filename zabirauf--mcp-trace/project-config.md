---
trigger: always_on
description: This file provides guidance to Claude Code when working with the MCP Proxy TUI project.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the MCP Proxy TUI project.
## Project Overview

This is a Terminal User Interface (TUI) application built with **Rust and Ratatui** that acts as a proxy and monitor for Model Context Protocol (MCP) server traffic. It consists of two separate binaries that work together to intercept, log, and display all communication between MCP clients and servers in real-time.

## Architecture

The project is structured as a **Cargo workspace** with two main components:

### 1. MCP Proxy (`mcp-proxy`)
- **Purpose**: STDIO-based proxy for MCP servers
- **Language**: Rust with tokio for async operations
- **Functionality**: Intercepts and forwards JSON-RPC communication, sends logs to monitor
- **Binary location**: `target/release/mcp-proxy`

### 2. MCP Monitor (`mcp-monitor`) 
- **Purpose**: TUI for monitoring multiple MCP proxies
- **Language**: Rust with Ratatui for terminal UI
- **Functionality**: Real-time display of logs, statistics, and proxy status
- **Binary location**: `target/release/mcp-monitor`

### 3. Shared Library (`mcp-common`)
- **Purpose**: Common types, IPC communication, and message protocols
- **Components**: Types, IPC (Unix sockets), JSON-RPC handling

```
MCP Client → [mcp-proxy] → MCP Server (STDIO)
                 ↓ (IPC)
           [mcp-monitor TUI]
                 ↓
           [Log Storage]
```

## Repository Information

**GitHub Repository**: `zabirauf/mcp-trace`

## Binary Distribution

When setting up binary distribution, **only the `mcp-trace` binary should be distributed**. The other binaries (`mcp-monitor` and `mcp-proxy`) are for internal use only and should not be included in public distributions.

## Commit Message Style

Keep git commit messages concise and direct. Avoid verbose explanations.

## Branch Management

For new tasks unrelated to the current work, always create a new branch from `main`:
```bash
git checkout main
git pull origin main
git checkout -b feature/new-task-name
```

## Key Components

### MCP Proxy Components
- **`src/main.rs`**: CLI argument parsing and application entry point
- **`src/proxy.rs`**: Main proxy logic and MCP server process management
- **`src/stdio_handler.rs`**: STDIO communication handling between client and server

### MCP Monitor Components  
- **`src/main.rs`**: TUI application setup and IPC server
- **`src/app.rs`**: Application state management and event handling
- **`src/ui.rs`**: Ratatui interface components and layout

### MCP Common Components
- **`src/types.rs`**: Shared data structures (ProxyId, LogEntry, ProxyStats, etc.)
- **`src/messages.rs`**: IPC message protocol definitions
- **`src/ipc.rs`**: Unix domain socket communication
- **`src/mcp.rs`**: JSON-RPC message parsing and handling

## Development Guidelines

### Building and Running

```bash
# Build all components
cargo build --release

# Run monitor (starts IPC server)
./target/release/mcp-monitor --verbose

# Run proxy (in another terminal)
./target/release/mcp-proxy --name "My Server" --command python server.py --verbose

# Or use convenience script
./run.sh monitor
./run.sh proxy python server.py
```

### Adding New Features

#### 1. New Proxy Features
- **STDIO handling**: Modify `mcp-proxy/src/stdio_handler.rs`
- **Process management**: Update `mcp-proxy/src/proxy.rs`
- **New CLI options**: Add to `mcp-proxy/src/main.rs` clap Args struct

#### 2. New Monitor Features
- **UI components**: Extend widgets in `mcp-monitor/src/ui.rs`
- **Application logic**: Update state management in `mcp-monitor/src/app.rs`
- **Event handling**: Add new AppEvent variants and handlers

#### 3. New Communication Features
- **Message types**: Add to `mcp-common/src/messages.rs` IpcMessage enum
- **Data structures**: Define in `mcp-common/src/types.rs`
- **Protocol changes**: Update IPC handling in `mcp-common/src/ipc.rs`

### Code Patterns

#### Async/Await Usage
```rust
// All network operations use tokio
async fn handle_communication(&mut self) -> Result<()> {
    tokio::select! {
        result = self.read_stdin() => { /* handle */ }
        result = self.read_stdout() => { /* handle */ }
        _ = self.shutdown_rx.recv() => break,
    }
}
```

#### Error Handling
```rust
// Use anyhow for error propagation
use anyhow::{Result, Context};

async fn start_server(&self) -> Result<()> {
    Command::new(&self.command[0])
        .spawn()
        .context("Failed to start MCP server")?;
}
```

#### IPC Communication
```rust
// Structured message sending
let message = IpcMessage::LogEntry(log_entry);
ipc_client.send(message).await?;
```

### Testing and Development

```bash
# Check compilation
cargo check

# Run with logging
RUST_LOG=debug ./target/release/mcp-monitor

# Docker development
./run.sh build
docker compose up --build

# View logs
./run.sh logs
```

## Common Tasks

### Adding a New Log Type

1. **Add to LogLevel enum** in `mcp-common/src/types.rs`:
```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum LogLevel {
    // ... existing levels
    NewLevel,
}
```

2. **Update UI colors** in `mcp-monitor/src/ui.rs`:
```rust
let level_color = match log.level {
    // ... existing colors
    LogLevel::NewLevel => Color::Purple,
};
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zabirauf/mcp-trace](https://github.com/zabirauf/mcp-trace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
