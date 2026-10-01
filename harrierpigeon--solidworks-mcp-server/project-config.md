---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SolidWorks MCP Server bridges Claude AI with SolidWorks CAD via the Model Context Protocol (MCP), enabling natural language creation and manipulation of 3D CAD models. Requires Windows + SolidWorks installed.

## Commands

```powershell
# Install dependencies
pip install -r requirements.txt

# Or install as an editable package (exposes `solidworks` and `server` for import)
pip install -e .

# Run all tests (closes open docs first, then runs all categories)
python test.py

# Interactive CLI test picker — select tests by number, range, or category
python test.py --gui

# Run a single category
python test.py --category "Sketch Tools"
python test.py --category "Feature Tools"
python test.py --category "Integration"

# Run a single test by name
python test.py --test basic_cube
python test.py --test sketch_line

# List all available tests
python test.py --list

# Run the server directly (normally launched by an MCP client such as Claude Desktop)
python server.py

# Dev server with hot reload (requires watchdog)
python dev_server.py

# Close all open SolidWorks documents (standalone utility)
python clean.py
```

## Claude Desktop Configuration

The server is registered in `%APPDATA%\Claude\claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "solidworks": {
      "command": "python",
      "args": ["C:\\path\\to\\solidworks-mcp\\server.py"]
    }
  }
}
```

## Architecture

```
server.py                         # MCP server: registers tools, routes calls via dispatch map
solidworks/
  __init__.py                     # Package exports for all modules
  connection.py                   # COM connection to SolidWorks, template discovery, part creation
  state_tracker.py                # Centralized state tracker: stable IDs for features, sketches, entities
  state_query.py                  # MCP tools for querying tracked state (get_state, get_entity, get_sketch_entities)
  sketching.py                    # 2D sketch tools, dimensioning, and constraints with spatial tracking
  modeling.py                     # Core modeling (new_part, extrusion, cut-extrusion, mass_properties, list_features)
  selection_helpers.py            # Shared selection utilities (edge, face, plane, feature, axis)
  features.py                     # Boss/Base features (revolve, sweep, loft, boundary boss)
  cut_features.py                 # Cut features (cut revolve, cut sweep, cut loft, boundary cut)
  applied_features.py             # Applied features (fillet, chamfer, shell, draft, rib, wrap, intersect)
  patterns.py                     # Patterns (linear pattern, circular pattern, mirror)
  hole_features.py                # Hole features (hole wizard, cosmetic thread)
  reference_geometry.py           # Reference geometry (ref plane, ref axis, ref point, coordinate system)
  geometry_query.py               # Geometry inspection (body info, faces, edges, face edges, vertices) with feature tagging
  document_manager.py             # Document lifecycle (save, open, activate, close, list, capture_views)
  assembly.py                     # Assembly tools (new assembly, insert component, mates, interference, assembly queries)
  configurations.py               # Configurations (variants, per-config dims) and equations
  feature_tree.py                 # Timeline as structured data, renaming, folders
  com_utils.py                    # Shared late-bound COM helpers (com_prop)
test.py                           # Unified test suite with registry, CLI selector (--gui), category/test filters
clean.py                          # Close all open SolidWorks documents (standalone utility)
scripts/install.ps1               # End-user installer: installs uv, downloads latest release, sets up venv, patches Claude Desktop config
```

**Test suite (`test.py`):** Uses a decorator-based test registry with 6 categories (Basic, Sketch Tools, Feature Tools, MCP Tools, Assembly, Integration). Each test is `def test_xxx(sw, template) -> bool`. The runner closes all open docs before starting, and between each test. The Integration category wraps the sequential cut-extrude reliability sub-tests as a single meta-test.

**Data flow:** Claude → MCP tool call → `server.py` → `_route_tool()` (dispatch map) → module → SolidWorks COM API

**Routing:** `server.py` builds a `{tool_name: module}` dispatch map at startup from each module's `get_tool_definitions()`. No static tool lists needed—adding a tool to a module auto-registers it. `server.py` itself defines one extra tool: `solidworks_batch` (up to 25 sequential tool calls in one MCP request; no result chaining; stops on first error unless `stopOnError=false`; cannot nest itself).

**Workflow order:** `new_part` → `create_sketch` → sketch entities → (optional: dimensions/constraints) → `exit_sketch` → feature creation (extrude, revolve, etc.)

## Key Implementation Details

**Units:** All tool inputs/outputs use millimeters. The SolidWorks COM API requires meters, so all values are divided by 1000 internally before API calls. Angles are input in degrees and converted to radians internally.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HarrierPigeon/Solidworks-MCP-Server](https://github.com/HarrierPigeon/Solidworks-MCP-Server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
