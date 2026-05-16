---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Install Dependencies
```bash
# Install package in development mode
pip install -e .

# Install with dev dependencies for testing and linting
pip install -e ".[dev]"
```

### Testing
```bash
# Run all tests
pytest tests/ -v

# Run specific test file
pytest tests/test_integration.py -v

# Run with coverage (if pytest-cov installed)
pytest tests/ -v --cov=kimai_mcp
```

### Code Formatting and Linting
```bash
# Format code with black
black src/ tests/

# Run linting with ruff
ruff check src/ tests/

# Fix linting issues automatically
ruff check --fix src/ tests/
```

### Running the Server

There are three server types for different use cases:

```bash
# 1. LOCAL MCP SERVER (for Claude Desktop)
python -m kimai_mcp --kimai-url=https://your-kimai.com --kimai-token=your-token
# or: kimai-mcp --kimai-url=... --kimai-token=...

# 2. SSE REMOTE SERVER (for Claude Desktop remote connections)
python -m kimai_mcp.sse_server --host 0.0.0.0 --port 8000
# or: kimai-mcp-server --host 0.0.0.0 --port 8000

# 3. STREAMABLE HTTP SERVER (for Claude.ai Connectors - NEW v2.8.0)
python -m kimai_mcp.streamable_http_server --users-config=./config/users.json
# or: kimai-mcp-streamable --users-config=./config/users.json
```

| Server | Command | Protocol | Use Case |
|--------|---------|----------|----------|
| Local | `kimai-mcp` | MCP Stdio | Claude Desktop local |
| SSE | `kimai-mcp-server` | HTTP/SSE | Claude Desktop remote |
| Streamable | `kimai-mcp-streamable` | HTTP Streamable | Claude.ai Connectors |

## Releasing a New Version

**CRITICAL: Always update version numbers in BOTH files before creating a release tag!**

### Version Files

| File                         | Line | Example                 |
|------------------------------|------|-------------------------|
| `pyproject.toml`             | 7    | `version = "2.11.2"`    |
| `src/kimai_mcp/__init__.py`  | 3    | `__version__ = "2.11.2"`|

### Release Steps

```bash
# 1. Update version in BOTH files (must match!)
# Edit pyproject.toml: version = "X.Y.Z"
# Edit src/kimai_mcp/__init__.py: __version__ = "X.Y.Z"

# 2. Commit version bump
git add pyproject.toml src/kimai_mcp/__init__.py
git commit -m "chore: Bump version to X.Y.Z"
git push origin main

# 3. Create and push tag
git tag vX.Y.Z
git push origin vX.Y.Z

# 4. Create GitHub Release from tag
# PyPI deployment triggers automatically via .github/workflows/publish.yml
```

### Common Pitfall
If PyPI deployment fails with "version already exists", the version numbers in the code files were not updated before tagging. Fix by updating both files, committing, and re-creating the release.

## Architecture Overview

### Core Components

1. **MCP Server (`server.py`)**: Main server implementation that handles MCP protocol communication and tool registration. **Now uses consolidated tools (10 tools instead of 73)** - 87% reduction while maintaining all functionality.

2. **Kimai API Client (`client.py`)**: HTTP client wrapper using httpx for all Kimai API interactions. Handles authentication, request formatting, and response parsing.

3. **Data Models (`models.py`)**: Pydantic models for type-safe data structures representing Kimai entities (timesheets, projects, users, etc.).

4. **Consolidated Tools (`tools/` directory)**: New consolidated tool implementations that replace 73 individual tools:
   - `entity_manager.py`: Universal CRUD operations for all entities (35 tools → 1)
   - `timesheet_consolidated.py`: All timesheet operations (9 tools → 1)
   - `timer_tool.py`: Timer management (4 tools → 1)
   - `rate_manager.py`: Rate management across entities (9 tools → 1)
   - `team_access_manager.py`: Team member and permission management (8 tools → 1)
   - `absence_manager.py`: Complete absence workflow (6 tools → 1)
   - `calendar_meta.py`: Calendar and meta field operations (6 tools → 3)
   - `project_analysis.py`: Advanced project analytics (kept as specialized tool)

### Key Design Patterns

1. **Action-Based Tools**: Tools use action parameters instead of separate tools (e.g., `entity` tool with `action: "create"` vs separate `create_entity` tool).

2. **Universal Entity Handler**: Single tool handles CRUD operations for all entity types using `type` and `action` parameters.

3. **Smart User Selection**: Tools like `timesheet` and `absence` implement intelligent user scope selection with `user_scope` enum ("self", "all", "specific").

4. **Consolidated Error Handling**: Unified error handling patterns across all consolidated tools.

5. **Flexible Configuration**: Supports CLI arguments, environment variables, and .env files.

### Authentication Flow
- API token passed via configuration
- Token included in all HTTP requests as X-AUTH-TOKEN header
- Optional default user ID for operations requiring user context

### Consolidated Tool Pattern
Each consolidated tool follows this structure:
1. Action routing based on `action` parameter
2. Input validation using Pydantic models
3. Entity-specific handler delegation (for entity tool)
4. API call through the Kimai client
5. Response transformation to MCP-compatible format

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [glazperle/kimai_mcp](https://github.com/glazperle/kimai_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
