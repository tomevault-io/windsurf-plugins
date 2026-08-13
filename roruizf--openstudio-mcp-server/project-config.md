---
trigger: always_on
description: **Purpose:** Guide AI assistants working with this Docker-based MCP server for OpenStudio building energy modeling.
---

# Claude AI Developer Guide - OpenStudio MCP Server

**Purpose:** Guide AI assistants working with this Docker-based MCP server for OpenStudio building energy modeling.

**Critical Context:** This server runs inside Docker to manage complex system-level OpenStudio dependencies (Python bindings, Ruby libraries, X11 libraries).

---

## Commands Reference

### Docker Build

Build the Docker image after changing dependencies or Dockerfile:

```bash
docker build -f .devcontainer/Dockerfile -t openstudio-mcp-dev .
```

**When to rebuild:**
- After modifying `pyproject.toml` dependencies
- After changing `.devcontainer/Dockerfile`
- After updating OpenStudio version

### Docker Run (Production)

Run the server in Docker as configured in Claude Desktop:

```bash
docker run --rm -i \
  -v C:\:/mnt/c \
  -v C:\openstudio-mcp-server:/workspace \
  -w /workspace/openstudio-mcp-server \
  openstudio-mcp-dev \
  uv run openstudio_mcp_server/server.py
```

**Volume Mounts:**
- `-v C:\:/mnt/c` - Entire C: drive access (enables smart path translation)
- `-v C:\openstudio-mcp-server:/workspace` - Server source code

### Local Debug (Inside Container)

For testing without Claude Desktop connection:

```bash
uv run openstudio_mcp_server/server.py
```

**Requirements:**
- Must run inside the Docker container (or with OpenStudio installed locally)
- Container must have access to OpenStudio Python bindings via `PYTHONPATH`

---

## Architecture & Components

### Overview

```
┌─────────────────────────────────────────────────────┐
│ Claude Desktop (Client)                             │
└────────────────┬────────────────────────────────────┘
                 │ MCP Protocol (stdio)
                 ↓
┌─────────────────────────────────────────────────────┐
│ Docker Container (openstudio-mcp-dev)               │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │ server.py (FastMCP Entry Point)              │  │
│  │  - Defines MCP tools                         │  │
│  │  - Handles client communication              │  │
│  │  - Routes requests to OpenStudioManager      │  │
│  └─────────────────┬────────────────────────────┘  │
│                    │                                │
│  ┌─────────────────▼────────────────────────────┐  │
│  │ openstudio_tools.py (Controller Layer)       │  │
│  │  - OpenStudioManager class                   │  │
│  │  - Minimal logic (validation, path resolution) │
│  │  - Calls openstudio_toolkit functions        │  │
│  │  - Formats responses as JSON                 │  │
│  └─────────────────┬────────────────────────────┘  │
│                    │                                │
│  ┌─────────────────▼────────────────────────────┐  │
│  │ openstudio_toolkit/ (Business Logic)         │  │
│  │  - Robust OpenStudio operations              │  │
│  │  - Direct interaction with OpenStudio API    │  │
│  │  - Pure functions (no state)                 │  │
│  │  - osm_objects/, tasks/, utils/              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  System Dependencies:                               │
│  - OpenStudio 3.7.0 (installed in /usr/local/)     │
│  - Python 3.12                                      │
│  - X11 libraries (for OpenStudio rendering)        │
└─────────────────────────────────────────────────────┘
```

### Component Details

#### 1. **server.py** (FastMCP Entry Point)

**Location:** `openstudio_mcp_server/server.py`

**Responsibilities:**
- Initializes FastMCP server
- Defines all MCP tools using `@mcp.tool()` decorator
- Handles MCP protocol communication (stdio)
- Routes tool calls to `OpenStudioManager`

**Example:**
```python
@mcp.tool()
async def load_osm_model(file_path: str, translate_version: bool = True) -> str:
    """Load an OpenStudio Model (OSM) file."""
    result = os_manager.load_osm_file(file_path, translate_version)
    return ensure_json_response(result)
```

**Key Point:** This layer should contain NO business logic - only tool definitions and routing.

---

#### 2. **openstudio_tools.py** (Controller/Bridge Layer)

**Location:** `openstudio_mcp_server/openstudio_tools.py`

**Responsibilities:**
- `OpenStudioManager` class maintains state (current loaded model)
- Input validation and path resolution
- Calls `openstudio_toolkit` functions
- Formats responses as dictionaries (converted to JSON by server.py)

**Design Principle:** **MINIMAL LOGIC**

This layer acts as a **thin controller** that:
1. Receives inputs from Claude (via server.py)
2. Validates inputs (e.g., "is a model loaded?")
3. Resolves file paths (using `path_utils.py`)
4. Calls robust functions from `openstudio_toolkit`
5. Returns simple structured responses

**Example:**
```python
def load_osm_file(self, file_path: str, translate_version: bool = True) -> Dict[str, Any]:
    # 1. Resolve path (with smart Windows→Docker translation)
    resolved_path = resolve_osm_path(self.config, file_path)

    # 2. Call toolkit function (where the real logic lives)
    from openstudio_toolkit.utils.osm_utils import load_osm_file_as_model
    model = load_osm_file_as_model(resolved_path, translate_version)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roruizf/openstudio-mcp-server](https://github.com/roruizf/openstudio-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
