---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

QGIS MCP (v0.2.0) connects QGIS to Claude AI through the Model Context Protocol (MCP), enabling Claude to directly control QGIS via socket-based communication. Includes a multi-client installer (`install.py`) for easy setup.

## Architecture

The system has two components that communicate over a TCP socket (default `localhost:9876`, configurable via env vars):

1. **QGIS Plugin** (`qgis_mcp_plugin/plugin.py`) — Runs inside QGIS (3.28–4.x). A `QgisMCPServer` class creates a non-blocking TCP socket server using a `QTimer` (25ms poll interval) to accept connections and process JSON commands within QGIS's event loop. Includes a `QgisMCPDockWidget` UI for start/stop control, and `QgisMCPPlugin` as the standard QGIS plugin entry point (`classFactory`). All command handlers live in this file. A companion `compat.py` module provides enum compatibility between QGIS 3.x and 4.x (see below).

2. **MCP Server** (`src/qgis_mcp/server.py`) — Runs outside QGIS as a standalone Python process. Uses `FastMCP` from the `mcp` library to expose QGIS operations as MCP tools, resources, and prompts. A `_send()` helper unwraps the response envelope and raises on errors. All 51 tools are `async` with `title=` for human-readable names. Uses `ToolAnnotations` for read-only/destructive/idempotent hints. Long-running tools use `ctx.info()` for MCP logging. Destructive tools use `ctx.elicit()` for user confirmation (with graceful fallback). An optional compound tool mode (`src/qgis_mcp/compound_tools.py`) groups tools into ~19 compound tools for reduced schema overhead.

**Data flow:** Claude → MCP Server (FastMCP) → TCP socket → QGIS Plugin (QTimer loop) → PyQGIS API → response back through socket.

There is also a standalone socket client at `src/qgis_mcp/client.py` (`QgisMCPClient` class) used for direct testing without MCP.

## Commands

```bash
# Run the MCP server (how Claude Desktop launches it)
uv run --no-sync src/qgis_mcp/server.py

# Run with custom host/port
QGIS_MCP_HOST=192.168.1.100 QGIS_MCP_PORT=9877 uv run --no-sync src/qgis_mcp/server.py

# Run with streamable HTTP transport (for remote/multi-client)
QGIS_MCP_TRANSPORT=streamable-http uv run --no-sync src/qgis_mcp/server.py

# Run with compound tool mode (reduces 51 tools to ~19 grouped tools)
QGIS_MCP_TOOL_MODE=compound uv run --no-sync src/qgis_mcp/server.py

# Run the multi-client installer (plugin symlink + MCP client config)
python install.py

# Run unit tests (no QGIS needed - mocked socket)
uv run --no-sync pytest tests/test_mcp_tools.py -v

# Run integration tests (requires QGIS plugin server running on localhost:9876)
uv run --no-sync pytest tests/test_qgis_live.py -v

# Run all tests
uv run --no-sync pytest tests/ -v
```

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `QGIS_MCP_HOST` | `localhost` | Host for QGIS plugin socket connection |
| `QGIS_MCP_PORT` | `9876` | Port for QGIS plugin socket connection |
| `QGIS_MCP_TRANSPORT` | `stdio` | MCP transport: `stdio` or `streamable-http` |
| `QGIS_MCP_LOG_FILE` | `~/.local/share/qgis-mcp/server.log` | Log file path (empty to disable file logging) |
| `QGIS_MCP_LOG_LEVEL` | `INFO` | File log level (`DEBUG`, `INFO`, `WARNING`, `ERROR`) |
| `QGIS_MCP_TOOL_MODE` | `granular` | Tool registration mode: `granular` (51 tools) or `compound` (~19 grouped tools) |

## MCP Tools (51 total)

| Tool | Title | Annotations | Description |
|---|---|---|---|
| `ping` | Ping | readOnly | Check server connectivity |
| `diagnose` | Diagnose | readOnly | Full stack health check: QGIS version, plugin/server version match, providers, clients |
| `get_qgis_info` | Get QGIS Info | readOnly | QGIS version, profile, plugins |
| `get_project_info` | Get Project Info | readOnly | Project metadata, CRS, layers |
| `load_project` | Load Project | — | Load a .qgs/.qgz file |
| `create_new_project` | Create New Project | — | Create and save new project |
| `save_project` | Save Project | idempotent | Save project to current or new path |
| `get_layers` | Get Layers | readOnly | List layers with pagination (limit/offset) |
| `add_vector_layer` | Add Vector Layer | — | Add vector layer (shapefile, GeoJSON, etc.) |
| `add_raster_layer` | Add Raster Layer | — | Add raster layer (GeoTIFF, etc.) |
| `remove_layer` | Remove Layer | destructive | Remove layer by ID (elicitation) |
| `find_layer` | Find Layer | readOnly | Find layers by name pattern (fnmatch/substring) |
| `create_memory_layer` | Create Memory Layer | — | Create in-memory vector layer with fields |
| `set_layer_visibility` | Set Layer Visibility | idempotent | Show/hide layer in layer tree |
| `zoom_to_layer` | Zoom to Layer | idempotent | Zoom canvas to layer extent |
| `get_layer_features` | Get Layer Features | readOnly | Flat features with _fid, expression filter, limit/offset, geometry |
| `get_field_statistics` | Get Field Statistics | readOnly | Aggregate stats for a field (count, mean, min, max, etc.) |
| `add_features` | Add Features | destructive | Add features to a vector layer |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nkarasiak/qgis-mcp](https://github.com/nkarasiak/qgis-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
