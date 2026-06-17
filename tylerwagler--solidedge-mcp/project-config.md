---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Solid Edge MCP (Model Context Protocol) server for AI-assisted CAD design. Windows-only, built on FastMCP and pywin32 COM automation. Licensed MIT.

The goal is to provide AI assistants with full access to Solid Edge CAD workflows: **connect → create → sketch → feature → query → export** with session management and undo/rollback support.

## Commands

```bash
# Install all dependencies (including dev)
uv sync --all-extras

# Run the MCP server (stdio transport)
uv run solidedge-mcp

# Run tests
uv run pytest
uv run pytest tests/unit/test_foo.py::test_bar  # single test

# Lint and format
uv run ruff check .
uv run ruff format .

# Type check
uv run mypy src/
```

## Architecture

### COM Automation Backend

Unlike KiCad (which uses file parsing), Solid Edge automation requires Windows COM through pywin32. The server communicates with a running Solid Edge instance via COM interfaces:

- **Connection layer** (`backends/connection.py`): Manages GetActiveObject/Dispatch, early/late binding
- **Document layer** (`backends/documents.py`): Create/open/save parts, assemblies, drafts
- **Sketching layer** (`backends/sketching.py`): 2D profile creation (lines, circles, arcs, rectangles, polygons)
- **Feature layer** (`backends/features/`): 3D operations (extrude, revolve, sweep, loft, holes, fillets) — mixin-based package
- **Assembly layer** (`backends/assembly.py`): Component placement, constraints, patterns
- **Query layer** (`backends/query.py`): Extract geometry, mass properties, feature trees
- **Export layer** (`backends/export.py`): Convert to STEP, STL, IGES, PDF, DXF

### Package Layout

```
src/solidedge_mcp/
├── server.py              # FastMCP server entry point
├── backends/              # COM automation implementations
│   ├── connection.py      # Application connection (GetActiveObject/Dispatch)
│   ├── documents.py       # Document create/open/save/close
│   ├── sketching.py       # 2D sketch profiles
│   ├── features/          # 3D feature operations (mixin-based package, 12 sub-modules)
│   ├── assembly.py        # Assembly operations
│   ├── query.py           # Model interrogation
│   ├── export.py          # Export to standard formats
│   └── constants.py       # Solid Edge API constants
├── tools/                 # MCP tool wrappers (~150 composite tools)
├── resources/             # MCP Resources (52 read-only endpoints)
├── prompts/               # MCP Prompt templates (pending)
└── session/               # Session/undo management (pending)
```

### Current State

**✅ FULLY IMPLEMENTED**: ~150 composite MCP tools + 52 MCP resources = ~200 total endpoints!

- **Backend layer**: Complete COM automation using pywin32 (connection, documents, sketching, features, assembly, query, export, diagnostics)
- **MCP tools**: ~150 composite tools registered via `tools/*.py` modules using `mcp.tool()` — each tool dispatches via a `method`/`type`/`action` discriminator parameter
- **MCP resources**: 52 read-only endpoints registered via `tools/resources.py` using `solidedge://` URIs
- **Coverage**: 96% of Solid Edge COM API methods implemented (394+ methods accessible via composite tools)
- **Test suite**: 1,380+ unit tests across 6 test files

**Pending**: Prompt templates, session management/undo

### Three-Pillar MCP Design

Following the MCP spec, the server exposes:

- **Tools** ✅ (~150 composite): Actions that create/modify models — each composite tool uses `method`/`type`/`action` discriminator to dispatch to the correct backend method
- **Resources** ✅ (52 implemented): Read-only model data (feature list, component tree, mass properties, document info, app info, sketch info)
- **Prompts** ⏳ (pending): Conversation templates (design review, manufacturability check, modeling guidance)

### Tool Categories (~150 tools + 52 resources)

| Category | Tools | Resources |
|---|---|---|
| **Connection/Application** | 12 | 0 |
| **Documents** | 9 | 0 |
| **Sketching** | 9 | 0 |
| **Features (Part)** | 58 | 0 |
| **Query/Analysis** | 19 | 52 |
| **Export/Drawing** | 28 | 0 |
| **Assembly** | 13 | 0 |
| **Diagnostics** | 2 | 0 |

### Tool Registration Pattern (Composite Tools)

Related backend operations are consolidated into **composite tools** that use a `method`/`type`/`action` discriminator parameter with `match/case` dispatch:

```python
def create_extrude(
    method: str = "finite",
    distance: float = 0.0,
    direction: str = "Normal",
    wall_thickness: float = 0.0,
    from_plane_index: int = 0,
    to_plane_index: int = 0,
) -> dict:
    """Create an extruded protrusion.

    method: 'finite' | 'infinite' | 'through_next' | 'from_to'
        | 'thin_wall' | 'symmetric' | ...
    """
    match method:
        case "finite":
            return feature_manager.create_extrude(distance, direction)
        case "infinite":
            return feature_manager.create_extrude_infinite(direction)
        # ... etc
        case _:
            return {"error": f"Unknown method: {method}"}
```

**Key patterns:**
- Each composite tool merges 2-18 related tools into one via discriminator parameter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tylerwagler/SolidEdge-MCP](https://github.com/tylerwagler/SolidEdge-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
