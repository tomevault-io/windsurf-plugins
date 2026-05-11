---
trigger: always_on
description: This document provides AI coding agents with project-specific guidance for the DelineaMCP (Model Context Protocol) server that integrates with Delinea Secret Server and Platform APIs.
---

# DelineaMCP Development Guide

This document provides AI coding agents with project-specific guidance for the DelineaMCP (Model Context Protocol) server that integrates with Delinea Secret Server and Platform APIs.

## Project Overview

DelineaMCP is an MCP server built on FastMCP that provides AI assistants with secure access to Delinea Secret Server and Platform APIs.
The system supports both STDIO transport (CLI tools like Claude Desktop) and SSE transport (web-based integrations like ChatGPT) with optional OAuth 2.0 authentication.

## Development Commands

**Package Manager**: Use `uv` as the primary interface for all Python operations.
This project heavily relies on uv for dependency management, virtual environment setup, and command execution.

### Setup and Installation

```bash
# Install dependencies (preferred)
uv sync --frozen

# Alternative dependency installation
uv pip sync requirements.txt
```

### Running the Server

```bash
# Development mode (STDIO transport)
uv run server.py

# With custom configuration
uv run server.py --config config.json

# SSE mode with OAuth (requires config)
uv run server.py --config production-config.json
```

### Testing

```bash
# Run unit tests (excludes integration tests)
uv run pytest -q --ignore=tests/integration/ tests/ -k "not test_live"

# Run with coverage
uv run coverage run -m pytest -q --ignore=tests/integration/ tests/ -k "not test_live"
uv run coverage report --omit="tests/*" --show-missing

# Run specific test file
uv run pytest tests/test_tools.py

# Integration tests (requires credentials)
export DELINEA_PASSWORD=<password>
export LIVE_SECRET_ID=<id>
uv run pytest tests/integration/
```

### Build Commands

```bash
# Compile dependencies (when updating pyproject.toml)
uv pip compile pyproject.toml --universal -o requirements.txt

# Lint check
uv run python -m py_compile server.py
uv run python -m py_compile delinea_api.py
```

### Linting && Formatting

Prior to any completion of work, ensure code linting and formatting is applied with the `trunk` tool.

You can install with: `curl https://get.trunk.io -fsSL | bash -s -- -y`.

For windows usage, this should be run in `git bash` or `wsl2` or it will fail.

```bash
trunk fmt && trunk check --fix
```

### Docker Development

```bash
# Build production image
docker build -t delinea-mcp:latest .

# Run with volume mounts for persistence
docker run --rm -p 8000:8000 \
  -e DELINEA_PASSWORD=<password> \
  -v $(pwd)/config.json:/app/config.json:ro \
  -v mcp-data:/app/data \
  delinea-mcp:latest

# Docker validation
docker build -t delinea-mcp:test .
docker run --rm delinea-mcp:test python server.py --help
```

## Architecture

DelineaMCP is an MCP server built on FastMCP that provides AI assistants with secure access to Delinea Secret Server and Platform APIs.

### Core Components

**Entry Point**: `server.py` - Thin wrapper that initializes the FastMCP server and registers all tools based on configuration

**Main Package**: `delinea_mcp/` contains:

- `tools.py` - Core MCP tools for Secret Server API interaction (secrets, folders, users, groups, roles)
- `user_platform_tools.py` - Delinea Platform user management tools (optional, requires platform configuration)
- `config.py` - Configuration loading utilities with environment variable fallbacks
- `constants.py` - Large string constants for SQL generation and reporting templates

**Authentication Module**: `delinea_mcp/auth/`

- `routes.py` - OAuth 2.0 endpoints for MCP client authentication
- `as_config.py` - Authorization server configuration and JWT handling
- `validators.py` - Token and request validation

**Transport Layer**: `delinea_mcp/transports/`

- `sse.py` - Server-Sent Events transport for HTTP-based clients

### Configuration System

Configuration is loaded from `config.json` in project root. The `config.py` module provides a centralized loader that falls back to environment variables for sensitive data.

Key configuration patterns:

- Non-secret values in `config.json` (usernames, URLs, feature flags)
- Sensitive data via environment variables (`DELINEA_PASSWORD`, `AZURE_OPENAI_KEY`)
- Transport mode selection (`stdio` vs `sse`)
- Authentication mode (`none` vs `oauth`)

### Transport Modes

**STDIO Mode**: Direct process communication for CLI tools like Claude Desktop

**SSE Mode**: HTTP Server-Sent Events for web-based integrations like ChatGPT

The server auto-detects transport mode from configuration and initializes the appropriate handler.

### Tool Registration

Tools are dynamically registered based on:

1. Available configuration (Azure OpenAI tools only register if API keys present)
2. `enabled_tools` configuration (empty list = all tools enabled)
3. Platform tools require platform-specific configuration

## Key Integration Points

### Secret Server API

Uses the `delinea_api.DelineaSession` class for authenticated API requests. The session handles bearer token management and automatic re-authentication.

### MCP Protocol

Built on `fastmcp.FastMCP` which provides the MCP server implementation. Tools are registered as async functions with type hints for automatic schema generation.

### OAuth Flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DelineaXPM/delinea-mcp](https://github.com/DelineaXPM/delinea-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
