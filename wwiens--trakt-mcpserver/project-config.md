---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

**Model Context Protocol (MCP) server** bridging AI models with Trakt.tv API. Built with FastMCP, providing entertainment data access with OAuth authentication.

**Trakt API Reference**: See `trakt.apib` for complete API documentation and endpoint specifications.

## Architecture - Single Responsibility Principle

**CRITICAL REQUIREMENT**: Every file must have exactly ONE clear purpose. No exceptions.

**BEFORE ADDING FEATURES**: Examine existing code patterns, design style, and architectural conventions to ensure consistency.

**Domain boundaries mirror `trakt.apib`**: Each top-level API resource (shows, movies, seasons, episodes, etc.) is its own domain — do NOT nest one domain's tools/clients inside another.

### Current Structure

```text
server/           # MCP server modules by domain
├── auth/         # Authentication tools/resources
├── shows/        # Show-specific tools/resources
├── seasons/      # Season-specific tools
├── movies/       # Movie-specific tools/resources
├── user/         # User data tools/resources
├── comments/     # Comment tools
├── search/       # Search tools
├── checkin/      # Check-in tools
└── prompts/      # MCP prompts

client/           # HTTP clients by domain
├── auth/         # Authentication client
├── shows/        # Show API client
├── seasons/      # Season API client
├── movies/       # Movie API client
├── comments/     # Comments API client
├── user/         # User API client
├── search/       # Search API client
└── checkin/      # Check-in API client

models/           # Pydantic models by domain
├── auth/         # Authentication models
├── formatters/   # Domain-specific formatters
├── movies/       # Movie models
├── shows/        # Show models
└── user/         # User models

config/           # Configuration by domain
├── auth/         # Auth constants
├── api/          # API constants  
├── endpoints/    # API endpoints by domain
└── mcp/          # MCP resource/tool definitions
    ├── resources/ # Domain-specific MCP resources
    └── tools/     # Domain-specific MCP tools

utils/api/        # API utilities
├── errors.py     # Error handling decorator
└── responses.py  # Response helpers
```

### Domain-Focused Imports

```python
# Use focused imports - import only what you need
from client.auth.client import AuthClient
from client.shows.client import ShowsClient
from client.movies.client import MoviesClient
from server.auth.tools import device_auth_start
from models.auth.auth import TraktAuthToken
from config.endpoints.shows import SHOWS_ENDPOINTS
```

## Development Commands

### Setup
```bash
pip install -e .
pip install -r requirements-dev.txt
cp .env.example .env
```

### Docker
```bash
# Build stdio image
docker build -f Dockerfile.stdio -t trakt_mcpserver:stdio .

# Run stdio container
docker run -i --rm --name trakt_mcpserver_stdio \
  -e TRAKT_CLIENT_ID=your_client_id \
  -e TRAKT_CLIENT_SECRET=your_client_secret \
  trakt_mcpserver:stdio

# Build SSE image
docker build -t trakt_mcpserver .

# Run SSE container
docker run -d --rm --name trakt_mcpserver \
  -e TRAKT_CLIENT_ID=your_client_id \
  -e TRAKT_CLIENT_SECRET=your_client_secret \
  -p 8080:8080 \
  trakt_mcpserver
```

### Testing & Quality
```bash
# Tests
pytest
pytest tests/client/auth/ -v  # Focused testing

# Code quality (zero tolerance for errors/warnings)
ruff check --fix    # Linting  
ruff format         # Formatting
pyright             # Type checking
pip-audit           # Security scanning
npx @modelcontextprotocol/inspector --cli python server.py --method tools/list    # MCP validation

# MCP server
python server.py                                                        # Direct run
mcp dev server.py                                                      # Development mode
npx @modelcontextprotocol/inspector --cli python server.py --method tools/list    # MCP validation (list tools)
npx @modelcontextprotocol/inspector --cli python server.py --method resources/list # MCP validation (list resources)
npx @modelcontextprotocol/inspector --cli python server.py --method prompts/list   # MCP validation (list prompts)
npx @modelcontextprotocol/inspector --cli python server.py --method tools/call     # Call specific tool
```

## Code Quality Requirements

### Single Responsibility Principle
- **Every file has exactly ONE purpose** - no mixed concerns
- **Every function has exactly ONE responsibility** 
- **Every class has exactly ONE reason to change**
- Files mixing domains are architectural violations

### Type Safety
- Type hints required for all functions/methods/attributes
- Use specific types: `list[str]` not `list`
- Return type annotations including `None`
- **CRITICAL: Prefer precise typing**: Use most specific types possible - NEVER use `Any` when concrete types are known
- **Avoid `cast()`**: Use type guards, unions, or proper design instead of bypassing type checker
- **Use `Final` for constants**: Module-level constants require `Final` type annotation

### Code Standards
- PEP 8 compliance
- 88 character line limit
- Descriptive names (no abbreviations)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wwiens/trakt_mcpserver](https://github.com/wwiens/trakt_mcpserver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
