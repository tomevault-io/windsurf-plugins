---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

IDA Pro MCP Server - enables LLM-assisted reverse engineering by bridging IDA Pro with Model Context Protocol clients through a JSON-RPC HTTP server.

**Architecture**: Multi-instance Gateway design
- **MCP Server** (`server.py`): Python >=3.11, runs via `uv`, proxies to Gateway
- **Gateway Server** (`gateway.py`): Routes requests to multiple IDA instances (port 13337)
- **IDA Plugin** (`ida_mcp.py`): Runs inside IDA Pro, registers with Gateway, exposes MCP server over HTTP (ports 13338+)

```
AI Client ──MCP──> Gateway (port 13337) ──> IDA Instance 1 (port 13338)
                                        ──> IDA Instance 2 (port 13339)
                                        ──> IDA Instance N (port 1333N+1)
```

## Development Commands

### Testing MCP Server
```bash
# Interactive MCP inspector (web UI at http://localhost:5173)
uv run mcp dev src/ida_pro_mcp/server.py
```

### Installation
```bash
# Install package + configure all MCP clients + install IDA plugin
pip install https://github.com/mrexodia/ida-pro-mcp/archive/refs/heads/main.zip
ida-pro-mcp --install

# Manual plugin install (creates symlinks or copies to IDA plugins folder)
uv run ida-pro-mcp --install

# Uninstall everything
uv run ida-pro-mcp --uninstall
```

### Running
```bash
# Standard stdio transport (used by most MCP clients)
uv run ida-pro-mcp

# SSE transport for headless/remote use
uv run ida-pro-mcp --transport http://127.0.0.1:8744/sse

# Headless mode with idalib (no GUI)
uv run idalib-mcp --host 127.0.0.1 --port 8745 path/to/binary

# Enable unsafe debugger functions
uv run ida-pro-mcp --unsafe
```

### Changelog Generation
```bash
# Direct commits to main since tag
git log --first-parent --no-merges 1.2.0..main "--pretty=- %s"
```

## Architecture Deep Dive

### Plugin Architecture (ida_mcp/)

**Modular API**: 9 specialized modules
- `api_core.py`: IDB metadata, function/string/import listing
- `api_analysis.py`: Decompilation, disassembly, xrefs, paths, patterns
- `api_memory.py`: Read bytes/integers/strings, patch operations
- `api_types.py`: Structures, type inference, type application
- `api_modify.py`: Comments, assembly patching, renaming
- `api_stack.py`: Stack frame operations
- `api_debug.py`: Debugger control (unsafe, requires `--unsafe` flag)
- `api_python.py`: Python code execution in IDA context
- `api_resources.py`: MCP resources (24 URI patterns for RESTful access via `ida://` URIs)

**Infrastructure**:
- `rpc.py`: JSON-RPC registry + type checking (`@tool`, `@resource`, `@unsafe` decorators)
- `sync.py`: IDA thread synchronization (`@idasync` decorator)
- `zeromcp/mcp.py`: HTTP/SSE server implementation (Streamable HTTP + SSE transports)
- `utils.py`: TypedDict schemas, address parsing, pagination helpers

**Vulnerability Scanning** (`api_vuln.py`):
- `vuln_scan()`: Scan binary for dangerous function calls
- `vuln_scan_details()`: Get detailed findings by category
- `vuln_scan_function()`: Scan specific function
- `vuln_categories()`: List supported vulnerability categories

### Decorator Chain Pattern

Every API function follows this pattern:
```python
@tool             # 1. Register MCP tool
@idasync          # 2. Execute on IDA's main thread
def my_api(param: Annotated[str, "description"]) -> ReturnType:
    """Docstring becomes MCP tool description"""
    # Implementation uses IDA SDK
```

### Thread Safety

**All IDA SDK calls MUST run on main thread** - enforced by `@idasync`:
- Use `@idasync` for all IDA SDK operations (both read and write)
- Implementation: `sync_wrapper()` uses `idaapi.execute_sync()` with queue-based result passing

### Type Annotations

**Batch-first API convention**: Most functions accept `str` (comma-separated) OR `list`:
```python
def my_api(addrs: Annotated[str, "Addresses (0x401000, 0x402000) or list"]):
    parsed = normalize_list_input(addrs)  # Handles both formats
```

**Annotated types**: Description text becomes MCP parameter description
```python
count: Annotated[int, "Maximum number of results"]
```

## Adding New API Functions

### Step-by-step

1. Choose the appropriate `api_*.py` file (or create new one following `api_*.py` pattern)
2. Import required IDA SDK modules and decorators:
   ```python
   from .rpc import tool
   from .sync import idasync
   ```
3. Define function with full type hints:
   ```python
   @tool
   @idasync
   def my_function(param: Annotated[str, "param description"]) -> dict:
       """Tool description (first line used in MCP schema)"""
       # Use IDA SDK here
       return {"result": value}
   ```
4. Test with MCP inspector: `uv run mcp dev src/ida_pro_mcp/server.py`

**No other changes needed** - AST parsing auto-discovers and registers the function.

### Unsafe Functions

Mark debugger operations or destructive actions as unsafe:
```python
@unsafe           # Requires --unsafe flag
@tool
@idasync
def dangerous_op():
    pass
```

### MCP Resources

Expose RESTful URI-based access to IDA data using `@resource`:
```python
@resource(uri="ida://functions/{pattern}")
@idasync
def functions_resource(pattern: str = "*") -> list[dict]:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QYmag1c/ida-pro-mcp-multi](https://github.com/QYmag1c/ida-pro-mcp-multi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
