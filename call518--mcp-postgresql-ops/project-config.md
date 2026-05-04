---
trigger: always_on
description: This is a **Model Context Protocol (MCP) server** built with **FastMCP** that provides PostgreSQL database monitoring and operations through natural language queries. The server acts as a safe, read-only bridge between AI assistants and PostgreSQL databases.
---

# MCP PostgreSQL Operations Server - AI Coding Agent Instructions

## Architecture Overview

This is a **Model Context Protocol (MCP) server** built with **FastMCP** that provides PostgreSQL database monitoring and operations through natural language queries. The server acts as a safe, read-only bridge between AI assistants and PostgreSQL databases.

### Core Components
- **`mcp_main.py`**: Main MCP server with 24+ `@mcp.tool()` decorated functions
- **`functions.py`**: Database connection layer using `asyncpg` with multi-database support
- **`version_compat.py`**: PostgreSQL 12-17 version detection and adaptive feature handling
- **`prompt_template.md`**: Comprehensive prompt definitions loaded via `@mcp.prompt()` decorators
- **Docker stack**: PostgreSQL + MCP server + MCPO proxy + Open WebUI integration

### Key Patterns

**Multi-Database Architecture**: All tools accept optional `database_name` parameter to target specific databases while maintaining a default connection database from `POSTGRES_DB` env var.

**Extension Dependencies**: Core functionality requires `pg_stat_statements` extension; `pg_stat_monitor` is optional. Always check extension availability with `check_extension_exists()` before using related tools.

**Version-Aware Tools**: Use `version_compat.py` for PostgreSQL 12-17 compatibility. Tools auto-adapt features based on detected version. **PostgreSQL 18 is beta and not yet supported** - will be added once stable.

**Tool Structure**: Each MCP tool follows this pattern:
```python
@mcp.tool()
async def get_something(limit: int = 20, database_name: str = None) -> str:
    """Detailed docstring with [Tool Purpose], [Exact Functionality], [Required Use Cases], [Strictly Prohibited Use Cases]"""
    try:
        # Validate inputs (limit constraints: max 1-100)
        # Check extension dependencies if needed
        # Execute queries via functions.py
        # Return formatted table data
    except Exception as e:
        logger.error(f"Failed to...: {e}")
        return f"Error: {str(e)}"
```

**Version Compatibility Pattern**: Critical for PostgreSQL 12 support - many tools use version-aware query builders:
```python
# In functions.py
from .version_compat import get_pg_stat_statements_query

async def get_pg_stat_statements_data(limit: int = 20, database: str = None):
    base_query = await get_pg_stat_statements_query(database)  # Auto-adapts for PG12/13+
    query = f"{base_query} LIMIT $1"
    return await execute_query(query, [limit], database=database)
```

## Development Workflows

### Local Development
```bash
# Primary development command - loads .env, starts MCP Inspector
./scripts/run-mcp-inspector-local.sh

# Direct execution for debugging with custom log levels
python -m src.mcp_postgresql_ops.mcp_main --log-level DEBUG --type streamable-http
```

### Docker Development
```bash
# Full stack with PostgreSQL + test data
docker-compose up -d

# Test data generation (creates 4 databases with 83k+ records)
./scripts/create-test-data.sh
```

### Environment Configuration
- Copy `.env.example` to `.env` and modify connection parameters
- **Critical**: `POSTGRES_DB` serves dual purpose - default connection target AND Docker database creation name
- Use `host.docker.internal` for `POSTGRES_HOST` when connecting from containers to host PostgreSQL

## Code Conventions

### Database Connection Pattern
```python
# Multi-database support - database parameter overrides POSTGRES_CONFIG default
async def get_db_connection(database: str = None) -> asyncpg.Connection:
    config = POSTGRES_CONFIG.copy()
    if database:
        config["database"] = database  # Override default
    return await asyncpg.connect(**config)
```

### Error Handling
- All MCP tools must return `str` (never raise exceptions to MCP layer)
- Log errors with `logger.error()` then return user-friendly error messages
- Mask sensitive information in connection info with `sanitize_connection_info()`

### Query Formatting
- Use `format_table_data(results, title)` for consistent table output
- Apply `format_bytes()` and `format_duration()` for human-readable values
- Enforce limit constraints: `limit = max(1, min(limit, 100))`

### Tool Compatibility Matrix
When adding new tools, **must** update the compatibility matrix in `README.md`:
- Classify as Extension-Independent, Version-Aware, or Extension-Dependent
- Document PostgreSQL version support (12-17)
- List system views/tables used
- Update tool count statistics

### Recent Major Changes
- **PostgreSQL 18 Support Removed**: Now supports 12-17 range (18 is beta)
- **Comprehensive PG12 Compatibility**: All tools now work on PostgreSQL 12
- **Version-Aware Query Generation**: Automatic column mapping and NULL handling
- **Enhanced Schema Analysis**: New tools for table relationships and schema introspection

## Project-Specific Integrations

### Prompt Template System
The server loads prompts from `prompt_template.md` using a custom parsing system:
- `@mcp.prompt("prompt_template_full")` - complete template
- `@mcp.prompt("prompt_template_headings")` - section list only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [call518/MCP-PostgreSQL-Ops](https://github.com/call518/MCP-PostgreSQL-Ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
