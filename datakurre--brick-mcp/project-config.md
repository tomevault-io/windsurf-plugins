---
trigger: always_on
description: Primary reference for AI coding agents (Copilot, Claude, etc.) working inside this repository. Read it before making any changes.
---

# AGENTS.md — brick-mcp developer guide for AI coding agents

Primary reference for AI coding agents (Copilot, Claude, etc.) working inside this repository. Read it before making any changes.

---

## What this project is

`brick-mcp` is a [Model Context Protocol](https://modelcontextprotocol.io/) (MCP) server that lets AI assistants create, inspect, and edit LEGO models stored in BrickLink Studio (`.io`) and LDraw (`.ldr`) files. It is built with [FastMCP](https://github.com/jlowin/fastmcp). There is no graphical output — all tools return structured JSON dicts.

---

## Repository layout

```
main.py                        Entry point — runs the MCP server over stdio
pyproject.toml                 Package metadata and dependencies
src/
  brick_mcp/
    __init__.py                Imports tools (side-effect: registers them) + re-exports mcp
    server.py                  FastMCP instance + INSTRUCTIONS string shown to the AI agent
    model.py                   StudioProject class, global singleton, get_model()/set_model()
    ldraw.py                   LDraw parser + serializer (type-1 and type-11)
    io_file.py                 .io ZIP read/write (plain + AES fallback)
    colors.py                  LDraw color code database
    _helpers.py                ok()/err()/ok_with_render()/try_render() helpers
    tools/
      __init__.py              Imports all tool sub-modules to trigger @mcp.tool() registration
      file_ops.py              new_model, open_model, save_model, get_model_info
      inspection.py            list_parts, get_bom, get_steps
      manipulation.py          add_part, remove_part, move_part, rotate_part,
                               change_color, add_step, remove_step
      parts.py                 search_parts, list_colors, get_color_info
      render.py                render_model (PNG rendering via LDView)
pkgs/
  ldview.nix                   Nix derivation for LDView headless renderer
tests/
  conftest.py                  Shared fixtures
  test_io_file.py              ZIP read/write tests
  test_ldraw.py                Parser/serializer tests
  test_model.py                StudioProject unit tests
  test_tools.py                MCP tool integration tests
```

---

## Architecture: the model singleton

`model.py` owns a **module-level singleton** of type `StudioProject`. Because `open_model` and `new_model` must replace it with a new object, the singleton is accessed through accessor functions — never through a direct import of the object:

```python
# model.py
def get_model() -> StudioProject: ...   # raises RuntimeError if no model loaded
def set_model(m: StudioProject | None) -> None: ...
```

**Every tool must call `get_model()` at invocation time.** Storing a reference at import time silently breaks after `open_model` is called.

```python
# CORRECT
from brick_mcp.model import get_model as _get_model
project = _get_model()

# WRONG — stale reference after open_model()
from brick_mcp.model import _model  # never do this
```

---

## The `StudioProject` class (`model.py`)

Holds a dict of `_SubmodelData` objects keyed by submodel name. The root submodel is identified by `project.root_submodel`.

### `_SubmodelData`

| Attribute | Type | Purpose |
|---|---|---|
| `name` | `str` | Submodel filename (e.g. `"model.ldr"`) |
| `commands` | `list[Command]` | Ordered list of `PartLine`, `MetaCommand`, `RawLine` |
| `_parts` | `dict[str, PartLine]` | UUID → PartLine index for O(1) lookup |

`_rebuild_index()` assigns UUIDs to all un-identified `PartLine`s on load.

### Key `StudioProject` methods

| Method | Notes |
|---|---|
| `add_part(submodel, part_number, color, x, y, z, matrix) → str` | Appends a PartLine; returns its UUID |
| `remove_part(part_id, submodel) → bool` | Deletes by UUID |
| `move_part(part_id, x, y, z, submodel) → bool` | Sets position in-place |
| `rotate_part(part_id, matrix, submodel) → bool` | Sets rotation matrix in-place |
| `change_color(part_id, color, submodel) → bool` | Sets color code in-place |
| `list_parts(submodel) → list[dict]` | Returns serializable part dicts |
| `get_bom(submodel) → dict` | `{part_number: {color_str: count}}` |
| `get_steps(submodel) → list[dict]` | Step boundaries with part counts |
| `to_ldraw_text() → str` | Serialize to standard LDraw (type-1) |
| `to_v2_ldraw_text() → str` | Serialize to BrickLink Studio v2 (type-11) |

---

## LDraw format (`ldraw.py`)

### Command types

| Class | Line type | Description |
|---|---|---|
| `PartLine` | `1` or `11` | Part placement with color, position, rotation, part file |
| `MetaCommand` | `0` | Comment or META (e.g. `STEP`, `FILE model.ldr`) |
| `RawLine` | `2`–`5` | Geometry primitives, passed through verbatim |

### BrickLink Studio type-11 extension

BrickLink Studio v2 writes `modelv2.ldr` using its own line type `11`:

```
11 <color> <uid> <selected> <group_id> <x> <y> <z> <9-matrix> <part_file>
```

The parser normalizes type-11 lines to `PartLine`, preserving `_uid`, `_group_id`, and `_selected` for round-trip fidelity. `to_v2_ldraw_text()` re-emits type-11 lines — preserving UIDs for existing parts, assigning new sequential UIDs for parts added since loading.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datakurre/brick-mcp](https://github.com/datakurre/brick-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
