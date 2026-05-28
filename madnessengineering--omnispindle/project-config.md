---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### 🚀 v1.0.0 Deployment Status (IMPORTANT!)

**Current Release**: v1.0.0 production-ready with comprehensive deployment modernization completed through Phase 6

**Completed Phases**:
- ✅ **Phase 1**: PM2 ecosystem modernized (Python 3.13, GitHub Actions, modern env vars)
- ✅ **Phase 2**: Docker infrastructure updated (Python 3.13, API-first, health checks)
- ✅ **Phase 3**: PyPI package preparation complete (build scripts, MANIFEST.in, entry points)
- ✅ **Phase 4**: Security review complete (git-secrets, credential audit, hardcoded IP cleanup)
- ✅ **Phase 6**: Documentation review (README.md updated, this CLAUDE.md refresh)

**Key Changes Made**:
- Modernized to Python 3.13 across all deployment configs
- Removed MongoDB dependencies from Docker (API-first architecture)
- Added comprehensive PyPI packaging with CLI entry points
- Implemented git-secrets protection with AWS patterns
- Enhanced .gitignore with comprehensive security patterns
- Updated all hardcoded IPs to use environment variables

**CLI Commands Available** (after `pip install omnispindle`):
- `omnispindle` - Web server for authenticated endpoints
- `omnispindle-server` - Alias for web server
- `omnispindle-stdio` - MCP stdio server for Claude Desktop

### Running the Server

**PyPI Installation (Recommended)**:
```bash
# Install from PyPI
pip install omnispindle

# Run MCP stdio server
omnispindle-stdio

# Run web server
omnispindle
```

**Development (Local)**:
```bash
# Run the stdio-based MCP server
python -m src.Omnispindle.stdio_server

# Run web server (Python 3.13 preferred)
python3.13 -m src.Omnispindle

# Using Makefile
make run  # Runs the server and publishes commit hash to MQTT
```

**Docker (Modernized)**:
```bash
# Build with modern Python 3.13 base
docker build -t omnispindle:v1.0.0 .

# Run with API-first configuration
docker run -e OMNISPINDLE_MODE=api omnispindle:v1.0.0
```

### PyPI Publishing

**Build and Test**:
```bash
# Use the build script
./build-and-publish-pypi.sh

# Manual build
python -m build
python -m twine check dist/*
```

**Publish**:
```bash
# Test PyPI
python -m twine upload --repository testpypi dist/*

# Production PyPI
python -m twine upload dist/*
```


## Architecture Overview

**Omnispindle v1.0.0** is a production-ready, API-first MCP server for todo and knowledge management. It serves as the coordination layer for the Madness Interactive ecosystem, providing standardized tools for AI agents through the Model Context Protocol.

### 🏗 Core Components (v1.0.0)

**MCP Server (`src/Omnispindle/`)**:
- `stdio_server.py` - Primary MCP server using FastMCP with stdio transport (CLI: `omnispindle-stdio`)
- `__main__.py` - CLI entry point and web server (CLI: `omnispindle`)
- `api_tools.py` - API-first implementation (recommended for production)
- `hybrid_tools.py` - Hybrid mode with API fallback (default mode)
- `tools.py` - Local database implementation (legacy mode)
- `api_client.py` - HTTP client for madnessinteractive.cc/api with JWT/API key auth
- `database.py` - MongoDB operations (hybrid/local modes only)
- `auth.py` - Authentication middleware with Auth0 integration
- `auth_setup.py` - Zero-config Auth0 device flow setup

**🔄 Operation Modes (Key Architecture Decision)**:
- **`api`** - Pure API mode, HTTP calls to madnessinteractive.cc/api (recommended)
- **`hybrid`** - API-first with MongoDB fallback (default, most reliable)
- **`local`** - Direct MongoDB connections only (legacy, local development)
- **`auto`** - Automatically choose best performing mode

**🔐 Authentication Layer**:
- **Zero-Config Auth**: Automatic Auth0 device flow with browser authentication
- **JWT Tokens**: Primary authentication method via Auth0
- **API Keys**: Alternative authentication for programmatic access (not implemented yet)
- **User Context Isolation**: All data scoped to authenticated user 

**📊 Data Layer**:
- **Primary**: madnessinteractive.cc/api (centralized, secure, multi-user)
- **Fallback**: Local MongoDB (todos, lessons, explanations, audit logs)
- **Real-time**: MQTT messaging for cross-system coordination
- **Collections**: todos, lessons, explanations, todo_logs (when using local storage)
- MQTT for real-time messaging and cross-system coordination

**Dashboard (`Todomill_projectorium/`)**:
- Node-RED based visual dashboard
- Real-time updates via MQTT
- JavaScript/HTML components in separate directories for version control

### MCP Tool Interface

**CRITICAL**: When integrating with HTTP MCP endpoints (for Inventorium chat, etc.), see integration standards in Inventorium's `docs/MCP_INTEGRATION_GUIDE.md`

**HTTP Endpoint Standards**:
- **URL**: `/api/mcp` (NOT `/mcp/` or `/mcp`)
- **Auth**: Use `get_current_user` dependency (header-based, NOT query param)
- **Context**: ALWAYS pass `ctx=Context(user=user)` to tools with Auth0 user
- **Never** use `Context(user=None)` - this breaks user database routing!

The server exposes standardized MCP tools that AI agents can call:

**Todo Management**:
- `add_todo` - Create new tasks with metadata (always include `metadata.files` for SwarmDesk 3D node linking)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MadnessEngineering/Omnispindle](https://github.com/MadnessEngineering/Omnispindle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
