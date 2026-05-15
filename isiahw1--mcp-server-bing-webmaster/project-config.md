---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that provides AI assistants with access to Bing Webmaster Tools API. It's a **hybrid npm/Python package** distributed via npm but implemented in Python, using a JavaScript wrapper (`run.js`) to spawn the Python process.

**Key Architecture Pattern**: The server acts as a bridge between MCP clients (Claude, Cursor, etc.) and Bing's REST API, using FastMCP to expose 60 Bing Webmaster Tools functions as MCP tools.

## Development Commands

### Setup & Installation
```bash
# Install dependencies (uses uv package manager)
make install
# OR
uv sync

# Install in editable mode for development
uv pip install -e .
```

### Code Quality
```bash
# Run type checking (mypy --strict) and linting (ruff)
make lint

# Format code with ruff
make format
```

### Testing
```bash
# Run all tests with pytest (requires BING_WEBMASTER_API_KEY)
make test

# Run tests directly with uv
uv run pytest mcp_server_bwt --doctest-modules
```

### Running the Server

```bash
# Start the server directly (for testing)
make start
# OR
uv run mcp_server_bwt/main.py
# OR
python -m mcp_server_bwt

# Run with MCP Inspector (for debugging MCP protocol)
make mcp_inspector
```

### Building & Distribution
```bash
# Build Python wheel
make build

# Build and test npm package
npm run build
npm run validate

# View what will be published
npm run view-package
```

### Versioning
```bash
# Sync version between Python (version.py) and npm (package.json)
npm run sync-version

# Version is stored in: mcp_server_bwt/version.py
```

## Architecture

### Dual Entry Points

**1. npm Distribution (`run.js`)**:
- Cross-platform JavaScript wrapper that spawns Python process
- Checks for virtual environment Python first, falls back to system Python
- Handles process lifecycle and signal forwarding
- This is what users execute: `npx @isiahw1/mcp-server-bing-webmaster`

**2. Python Module (`mcp_server_bwt/`)**:
- `main.py` (1585 lines): Core implementation with all MCP tools
- `__main__.py`: Python module entry point
- `version.py`: Single source of truth for version (synced to package.json)

### Core Components

**BingWebmasterAPI Class** (`main.py:35-121`):
- Async HTTP client wrapper using `httpx`
- Implements context manager pattern for resource management
- `_make_request()`: Handles all API communication
- `_ensure_type_field()`: Adds OData type metadata for MCP compatibility
- Base URL: `https://ssl.bing.com/webmaster/api.svc/json`

**MCP Tool Pattern** (`main.py:129-1575`):
All tools follow this structure:
```python
@mcp.tool(name="tool_name", description="...")
async def tool_name(param: Annotated[type, "description"]) -> ReturnType:
    async with api:
        result = await api._make_request("EndpointName", ...)
        return api._ensure_type_field(result, "TypeName")
```

**Server Initialization** (`main.py:20-25, 1578-1585`):
```python
mcp = FastMCP("mcp-server-bing-webmaster", version="1.0.0")
api = BingWebmasterAPI(API_KEY)

def app() -> None:
    mcp.run(transport="stdio")  # Uses stdio transport for MCP protocol
```

### Tool Organization

The 60 tools are organized into 16 functional categories:
- Site Management (4 tools)
- Traffic Analysis (6 tools)
- Crawling & Indexing (5 tools)
- URL Management (6 tools)
- Content Management (2 tools)
- Sitemap & Feed Management (5 tools)
- Keyword Analysis (6 tools)
- Link Analysis (3 tools)
- URL Blocking (3 tools)
- Deep Link Management (3 tools)
- Page Preview Management (3 tools)
- URL Query Parameters (4 tools)
- Geographic Settings (2 tools)
- Site Roles & Access (3 tools)
- Site Moves/Migration (2 tools)
- Child URLs (2 tools)

### Error Handling Pattern

All tools follow consistent error handling:
1. HTTP status code checking (raises on non-200)
2. Timeout exception handling (30s default)
3. Logging errors with context
4. Re-raising exceptions for MCP client handling

### API Response Pattern

Bing API returns OData-formatted responses:
- Response format: `{"d": {...actual data...}}`
- `_make_request()` automatically unwraps `data["d"]`
- `_ensure_type_field()` adds `__type` metadata for MCP compatibility

## Environment Configuration

**Required**:
- `BING_WEBMASTER_API_KEY`: Bing Webmaster Tools API key (obtained from Bing Webmaster Tools → Settings → API Access)

**Loading**: Environment variables are loaded at module level (`main.py:29`) and raise `ValueError` if missing.

## Adding New Tools

When adding new Bing Webmaster Tools endpoints:

1. **Check Bing API Documentation**: Verify the endpoint, method, and parameters
2. **Add tool definition** in `main.py` following the pattern:
   ```python
   @mcp.tool(name="tool_name", description="Clear description of what it does")
   async def tool_name(
       param1: Annotated[str, "Parameter description"],
       param2: Annotated[Optional[int], "Optional parameter"] = None,
   ) -> ReturnType:
       """Docstring with detailed explanation."""
       async with api:
           result = await api._make_request(
               "BingEndpointName",
               method="GET",  # or POST, PUT, DELETE

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isiahw1/mcp-server-bing-webmaster](https://github.com/isiahw1/mcp-server-bing-webmaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
