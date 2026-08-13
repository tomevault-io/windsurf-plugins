---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

lvkit converts LabVIEW VI files to Python code without requiring a LabVIEW license. It uses [pylabview](https://github.com/mefistotelis/pylabview) as the core parser for reading VI file formats.

## ⛔ CLEAN-ROOM — NEVER SUGGEST OPENING LabVIEW (READ THIS FIRST)

**The maintainer does NOT have LabVIEW installed and legally CANNOT use it for this work.** NI's EULA prohibits using LabVIEW to reverse-engineer LabVIEW; doing so would poison lvkit's clean room. **The clean room is the entire reason this project exists** — if LabVIEW were an option there would be no project.

So: **NEVER tell the maintainer to "open it in LabVIEW", "click the node", "check quick help / the manual in LabVIEW", or confirm anything by inspecting LabVIEW.** Not once. Identifying a primitive, a vilib VI, a type, or any behavior draws ONLY from clean-room sources:

1. **The parsed graph** — pylabview reads the `.vi` binary with no license: real per-terminal types (incl. array *element* type and refnum `ref_type`), dataflow, structure, constants.
2. **NI's PUBLIC web docs** — `https://docs-be.ni.com/api/bundle/labview-api-ref/page/...` (JSON `topic_html`; the `www.ni.com` SPA returns only a shell).
3. **Algorithm knowledge** (LZW, MD5, GIF, …) — reconstruct the VI's math and let it force the answer.
4. **Deterministic deduction** from 1–3 (e.g. "an output compared against a code *count* must be count-scaled → `2^width` → Power Of 2, not log2").

When stuck: do MORE of 1–4, or write a `"placeholder": true` primitive entry. Ask the maintainer to describe the *algorithm/domain* if needed — **never to inspect LabVIEW.** Also: ship ZERO NI-derived artwork (clean-room glyphs only).

## Commands

Always use `uv run` — it automatically activates the project venv without a separate activation step.

```bash
# Install with dev dependencies
uv sync

# Run all tests
uv run pytest

# Run a single test
uv run pytest tests/test_parser.py::test_parse_vi

# Lint
uv run ruff check .

# Type check
uv run python -m pyright src/

# Scripts
uv run python scripts/generate_python.py "path/to/file.vi" -o outputs
```

## Architecture

The conversion pipeline:

1. **Binary extraction**: pylabview (subprocess) reads the VI binary → XML files (`_BDHb.xml`, `_FPHb.xml`, `.xml`)
2. **Parsing** (`parser/`): `parse_vi()` converts XML → `ParsedVI` dataclasses (nodes, wires, constants, types, front panel)
3. **Graph construction** (`graph/`): `ParsedVI` → `InMemoryVIGraph` NetworkX multi-digraph. `get_vi_context()` returns `VIContext`.
4. **Code generation** (`codegen/`): `build_module(vi_context, vi_name)` walks `VIContext` → Python `ast.Module` → source string
5. **Orchestration** (`pipeline.py`): multi-VI load ordering, dependency resolution, file output

### Key Modules

- `src/lvkit/parser/` — XML → `ParsedVI` dataclasses (nodes, wires, constants, types)
- `src/lvkit/graph/` — `InMemoryVIGraph`, graph construction, queries, operations
- `src/lvkit/models.py` — shared type definitions used by parser, graph, and codegen (`LVType`, `Operation`, `Frame`, `Terminal`, `Tunnel`, etc.)
- `src/lvkit/graph/models.py` — graph/codegen-only types (`GraphNode` hierarchy, `VIContext`, `Wire`, query/info types, `BranchPoint`)
- `src/lvkit/codegen/builder.py` — `build_module()` entry point for AST generation
- `src/lvkit/pipeline.py` — orchestrates multi-VI generation
- `src/lvkit/cli.py` — command-line interface
- `src/lvkit/mcp/` — MCP server (16 tools)

### Standard Test Command

```bash
# Single VI
python scripts/generate_python.py "path/to/file.vi" -o outputs --search-path .lvkit/cache/samples/OpenG/extracted

# LabVIEW class (.lvclass)
python scripts/generate_python.py "path/to/MyClass.lvclass" -o outputs --search-path .lvkit/cache/samples/OpenG/extracted

# LabVIEW library (.lvlib)
python scripts/generate_python.py "path/to/MyLib.lvlib" -o outputs --search-path .lvkit/cache/samples/OpenG/extracted

# Directory of VIs
python scripts/generate_python.py "path/to/vi_folder/" -o outputs --search-path .lvkit/cache/samples/OpenG/extracted
```

## Error Handling Strategy

LabVIEW uses error clusters passed through wires. Python uses exceptions. The conversion strategy:

1. **No error clusters → Natural Python exceptions** - Just let exceptions propagate
2. **Error clusters + parallel branches → Held error model**

### Held Error Model

When a VI has parallel branches AND error terminals, we use this pattern:

```python
def my_vi(input_data):
    _held_error = None  # Track errors from branches

    # Parallel branch 0
    try:
        branch_0_result = branch_0_operations()
    except LabVIEWError as e:
        _held_error = _held_error or e
        branch_0_result = None

    # Parallel branch 1
    try:
        branch_1_result = branch_1_operations()
    except LabVIEWError as e:
        _held_error = _held_error or e
        branch_1_result = None

    # Merge point - raise first error
    if _held_error:
        raise _held_error

    return result
```

This preserves LabVIEW's semantics where:
- Branches continue executing even if one errors
- First error is preserved and raised at merge point

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pragmatest-dev/lvkit](https://github.com/pragmatest-dev/lvkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
