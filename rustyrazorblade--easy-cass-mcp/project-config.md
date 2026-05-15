---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Apache Cassandra MCP (Model Context Protocol) server built using FastMCP, a Python framework for creating MCP servers with streamable HTTP support. The server MUST use FastMCP's streamable HTTP transport for all communications.

Use the following reference: https://gofastmcp.com/getting-started/welcome

## Development Commands

### Package Management
This project uses `uv` for Python package management:
- `uv sync` - Sync all dependencies from pyproject.toml
- `uv add <package>` - Add a new dependency
- `uv add --dev <package>` - Add a new development dependency
- `uv lock` - Update the lock file

### Running the Server
- `python main.py` - Run the MCP server
- `uv run python main.py` - Run using uv's Python environment

### Installing for Claude Desktop
The MCP server can be installed for use with Claude Desktop using FastMCP:

1. **Start the MCP server** (runs on port 8000):
   ```bash
   uv run python main.py
   ```

2. **Install the proxy for Claude Desktop** (in another terminal):
   ```bash
   fastmcp install claude-desktop ecm/proxy.py:proxy
   ```

The proxy (`ecm/proxy.py`) connects Claude Desktop to the HTTP server running on port 8000. Both the server and proxy need to be running for Claude Desktop integration to work.

### Development Tools
- `black .` - Format code using Black
- `isort .` - Sort imports
- `flake8 .` - Run linting checks
- `mypy .` - Run type checking
- `pytest` - Run tests (when tests are added)
- `pytest -v` - Run tests with verbose output
- `pytest tests/test_specific.py::test_function` - Run a specific test

### Running All Checks
```bash
# Format and lint
black . && isort . && flake8 . && mypy .

# Run tests
pytest
```

## Project Architecture

### Core Dependencies
- **FastMCP (>=2.10.6)**: Framework for building MCP servers with streamable HTTP support
- **httpx (>=0.28.1)**: HTTP client library for making requests to Cassandra
- **Pydantic (>=2.11.7)**: Data validation and settings management

### Development Dependencies
- **black**: Code formatting
- **flake8**: Linting
- **isort**: Import sorting
- **mypy**: Type checking
- **pytest**: Testing framework
- **pytest-asyncio**: Async test support

### MCP Server Structure
When implementing the Cassandra MCP server, follow the FastMCP patterns:

1. **Server Definition**: Use FastMCP decorators to define server metadata and tools
2. **Tool Implementation**: Create tools for Cassandra operations (health checks, metrics, maintenance tasks)
3. **Async Support**: Use async/await for Cassandra operations to handle concurrent requests
4. **Error Handling**: Implement proper error handling for Cassandra connection issues

### Typical MCP Server Pattern
```python
from fastmcp import FastMCP

mcp = FastMCP("easy-cass-mcp")

@mcp.tool()
async def cassandra_health():
    """Check Cassandra cluster health"""
    # Implementation here
    pass
```

### FastMCP Async Usage
FastMCP provides both synchronous and asynchronous APIs:
- Use `mcp.run()` in synchronous contexts (regular functions)
- Use `await mcp.run_async()` in async contexts (async functions)

**Important**: The `run()` method cannot be called from inside an async function because it creates its own async event loop. Always use `run_async()` inside async functions.

Example async pattern:
```python
import asyncio
from fastmcp import FastMCP

mcp = FastMCP(name="MyServer")

async def main():
    # Setup any async resources
    await setup_connections()
    
    # Use run_async() in async contexts
    await mcp.run_async(transport="http")

if __name__ == "__main__":
    asyncio.run(main())
```

## Cassandra Integration 

All interactions should be handled through CQL and virtual tables.

### Using Abstractions

When implementing MCP tools, always favor using abstractions over raw Cassandra calls:

1. **CassandraUtility**: Use this for cluster-wide operations
   - `get_version()`: Always use this instead of manually querying system.local for version
   - `get_table(keyspace, table)`: Use this to create CassandraTable objects
   - The utility first checks if version is available in the driver metadata before falling back to system tables

2. **CassandraTable**: Use this for table-specific operations
   - `get_compaction_strategy()`: Returns compaction strategy class and options
   - `get_create_statement()`: Returns the CREATE TABLE statement
   - Provides a clean abstraction over raw metadata access

Example usage in MCP tools:
```python
# Good - using abstractions
utility = CassandraUtility(session)
version = await utility.get_version()
table = utility.get_table(keyspace, table_name)
compaction = await table.get_compaction_strategy()

# Avoid - raw queries for metadata
result = await session.execute("SELECT release_version FROM system.local")
```

### Node-Specific Queries

The MCP server supports executing queries on specific nodes or all nodes in the cluster. This is essential for querying virtual tables and node-local system tables.

#### Available MCP Tools for Node-Specific Queries:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustyrazorblade/easy-cass-mcp](https://github.com/rustyrazorblade/easy-cass-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
