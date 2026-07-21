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
- `UNRAID_MCP_MAX_RESPONSE_BYTES`: Max serialized tool-response size in bytes (default: 40000 = 40 KB ≈ 10K tokens). Responses over the cap are replaced with a structured, parseable JSON truncation marker (`{"error": "response_truncated", "truncated": true, ...}`) rather than hard-cut mid-JSON. This is a backstop; the per-list `cap_list` defaults do the primary bounding. See `unraid_mcp/core/response_limit.py`.

**SSL/TLS:**
- `UNRAID_VERIFY_SSL`: SSL verification (default: true; set `false` for self-signed certs)

**Subscriptions:**
- `UNRAID_AUTO_START_SUBSCRIPTIONS`: Lazily initialize enabled live subscriptions on first MCP resource/diagnostic access (default: true); the first read may return `connecting`
- `UNRAID_MAX_RECONNECT_ATTEMPTS`: WebSocket reconnect limit (default: 10)
- `UNRAID_MCP_ENABLE_RAW_SUBSCRIPTION_PROBE`: Debug-only, data-sensitive raw frame in `subscriptions/test_query` (default: false; never enable on shared deployments)
- Collection safety: `UNRAID_SUBSCRIPTION_COLLECT_MAX_EVENTS=100`, `UNRAID_SUBSCRIPTION_COLLECT_MAX_BYTES=1048576`, and `UNRAID_SUBSCRIPTION_COLLECT_MAX_SECONDS=30` bound retention while streaming; positive `limit` and the response-size budget may lower those ceilings
- Cache/timeout safety: cached subscription payloads are usable for at most `UNRAID_SUBSCRIPTION_CACHE_MAX_AGE_SECONDS=300`; per-call WebSocket timeout is capped by `UNRAID_SUBSCRIPTION_TIMEOUT_MAX_SECONDS=60`

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
- **List Capping**: List subactions bound output via `cap_list` (`core/pagination.py`) threaded
  from the `limit` tool param. Capped responses carry a `page` meta dict
  (`returned`/`total`/`truncated`, plus a `hint` when truncated). `limit<=0` returns everything;
  omitting `limit` uses the tool default (20). Every list-shaped subaction returns a capped
  collection plus `page` metadata, including VM, disk/log, key/permission, OIDC, system,
  Docker, array, notification, plugin, and live event/interface lists. Collect-mode live
  calls retain bounded events/bytes/duration even when `limit<=0`. When a live runtime
  safety cap fires, `page.truncated` remains true and `total_is_lower_bound` identifies the
  observed minimum rather than claiming a complete window. `docker/details` fetches a
  single container via `docker.container(id:)` rather than over-fetching the full container
  list.
- **Destructive Action Safety**: `DESTRUCTIVE_ACTIONS` sets require `confirm=True` for dangerous operations
- **Modular Architecture**: Clean separation of concerns across focused modules
- **Error Handling**: Uses ToolError for user-facing errors, detailed logging for debugging

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dinglebear-ai/unraid-mcp](https://github.com/dinglebear-ai/unraid-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
