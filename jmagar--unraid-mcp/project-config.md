---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
This is an MCP (Model Context Protocol) server that provides tools to interact with an Unraid server's GraphQL API. The server is built using FastMCP with a **modular architecture** consisting of separate packages for configuration, core functionality, subscriptions, and tools.

## Development Commands

### Setup
```bash
# Initialize uv virtual environment and install dependencies
uv sync

# Install dev dependencies
uv sync --group dev
```

### Running the Server
```bash
# Local development with uv (recommended)
uv run unraid-mcp-server

# Direct module execution
uv run -m unraid_mcp.main
```

### Code Quality
```bash
# Lint and format with ruff
uv run ruff check unraid_mcp/
uv run ruff format unraid_mcp/

# Type checking with ty (Astral's fast type checker)
uv run ty check unraid_mcp/

# Run tests
uv run pytest
```

### Environment Setup
Copy `.env.example` to `.env` and configure:

**Required:**
- `UNRAID_API_URL`: Unraid GraphQL endpoint
- `UNRAID_API_KEY`: Unraid API key

**Server:**
- `UNRAID_MCP_LOG_LEVEL`: Log verbosity (default: INFO)
- `UNRAID_MCP_LOG_FILE`: Log filename in logs/ (default: unraid-mcp.log)

**SSL/TLS:**
- `UNRAID_VERIFY_SSL`: SSL verification (default: true; set `false` for self-signed certs)

**Subscriptions:**
- `UNRAID_AUTO_START_SUBSCRIPTIONS`: Auto-start live subscriptions on startup (default: true)
- `UNRAID_MAX_RECONNECT_ATTEMPTS`: WebSocket reconnect limit (default: 10)

**Credentials override:**
- `UNRAID_CREDENTIALS_DIR`: Override the `~/.unraid-mcp/` credentials directory path

## Architecture

### Core Components
- **Main Server**: `unraid_mcp/server.py` - Modular MCP server with FastMCP integration
- **Entry Point**: `unraid_mcp/main.py` - Application entry point and startup logic
- **Configuration**: `unraid_mcp/config/` - Settings management and logging configuration
- **Core Infrastructure**: `unraid_mcp/core/` - GraphQL client, exceptions, and shared types
  - `guards.py` — destructive action gating via MCP elicitation
  - `utils.py` — shared helpers (`safe_get`, `safe_display_url`, path validation)
  - `setup.py` — elicitation-based credential setup flow
- **Subscriptions**: `unraid_mcp/subscriptions/` - Real-time WebSocket subscriptions and diagnostics
- **Tools**: `unraid_mcp/tools/` - Domain-specific tool implementations
- **GraphQL Client**: Uses httpx for async HTTP requests to Unraid API
- **Version Helper**: `unraid_mcp/version.py` - Reads version from package metadata via importlib

### Key Design Patterns
- **Consolidated Action Pattern**: Each tool uses `action: Literal[...]` parameter to expose multiple operations via a single MCP tool, reducing context window usage
- **Pre-built Query Dicts**: `QUERIES` and `MUTATIONS` dicts prevent GraphQL injection and organize operations
- **Destructive Action Safety**: `DESTRUCTIVE_ACTIONS` sets require `confirm=True` for dangerous operations
- **Modular Architecture**: Clean separation of concerns across focused modules
- **Error Handling**: Uses ToolError for user-facing errors, detailed logging for debugging
- **Timeout Management**: Custom timeout configurations for different query types (90s for disk ops)
- **Data Processing**: Tools return both human-readable summaries and detailed raw data
- **Health Monitoring**: Comprehensive health check tool for system monitoring
- **Real-time Subscriptions**: WebSocket-based live data streaming
- **Persistent Subscription Manager**: `live` action subactions use a shared `SubscriptionManager`
  that maintains persistent WebSocket connections. Resources serve cached data via
  `subscription_manager.get_resource_data(action)`. A "connecting" placeholder is returned
  while the subscription starts — callers should retry in a moment. When
  `UNRAID_AUTO_START_SUBSCRIPTIONS=false`, resources fall back to on-demand `subscribe_once`.

### Tool Categories (3 Tools: 1 Primary + 2 Diagnostic)

The server registers **3 MCP tools**:
- **`unraid`** — primary tool with `action` (domain) + `subaction` (operation) routing, 107 subactions. Call it as `unraid(action="docker", subaction="list")`.
- **`diagnose_subscriptions`** — inspect subscription connection states, errors, and WebSocket URLs.
- **`test_subscription_query`** — test a specific GraphQL subscription query (allowlisted fields only).

| action | subactions |
|--------|-----------|
| **system** (18) | overview, array, network, registration, variables, metrics, services, display, config, online, owner, settings, server, servers, flash, ups_devices, ups_device, ups_config |
| **health** (4) | check, test_connection, diagnose, setup |
| **array** (13) | parity_status, parity_history, parity_start, parity_pause, parity_resume, parity_cancel, start_array, stop_array*, add_disk, remove_disk*, mount_disk, unmount_disk, clear_disk_stats* |
| **disk** (6) | shares, disks, disk_details, log_files, logs, flash_backup* |
| **docker** (7) | list, details, start, stop, restart, networks, network_details |
| **vm** (9) | list, details, start, stop, pause, resume, force_stop*, reboot, reset* |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmagar/unraid-mcp](https://github.com/jmagar/unraid-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
