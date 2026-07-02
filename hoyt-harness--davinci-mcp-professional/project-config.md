---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

DaVinci MCP Professional is an enterprise-grade Model Context Protocol (MCP)
server that exposes DaVinci Resolve's full Python scripting API to AI assistants
(Claude Desktop, Cursor) via the MCP. It targets both Windows and macOS and
requires Python >= 3.10.

This is a hard/project fork from https://github.com/samuelgursky/davinci-resolve-mcp,
now independent due to major architectural overhaul. Licensed under GPL-3.0
(see COPYING.md).

## Development Environment

### Package Management

Use **uv** exclusively for dependency and virtual environment management.
Never use raw pip for project dependencies.

```bash
# First-time setup
uv venv
uv sync

# Run the server
uv run python main.py

# Run the MCP server entry point (for Claude Desktop / Cursor)
uv run python mcp_server.py

# Add a dependency
uv add <package>

# Upgrade a dependency
uv lock --upgrade-package <package>
```

### Python Version

Python >= 3.10 is required (the MCP Python SDK uses `match` statements and
other 3.10+ features).  All tool configurations (pyright, mypy, ruff)
must target 3.10 as the minimum version.

On **Windows**, the virtual environment MUST be created from the
system-installed Python — the same installation that appears in the
Windows registry under `HKLM\SOFTWARE\Python\PythonCore`.  DaVinci
Resolve's `fusionscript.dll` discovers Python via this registry key and
loads `python3.dll` by full path.  If the running Python is a different
installation (e.g. a uv-managed download), two Python runtimes end up
in the same process and the server crashes.  Use:

```bash
uv venv --python "C:\Program Files\PythonXYZ\python.exe"
uv sync
```

where `PythonXYZ` matches the system-installed version.

### Upstream Reference

The MCP Python SDK is the upstream dependency for protocol implementation.
A local reference copy is available at:

    D:\dev\ARTIFICIAL_INTELLIGENCE\MCP\_MCP-Tools-Dev\python-sdk

The upstream MCP specification repository is at:

    D:\dev\ARTIFICIAL_INTELLIGENCE\MCP\_MCP-Tools-Dev\modelcontextprotocol

Audit this project's MCP usage and dependency versions against these
references when making protocol-level changes.

## Common Commands

```bash
# Lint and format
uv run ruff check src/ tests/
uv run ruff format src/ tests/

# Type checking (pyright scoped to src/ per pyproject.toml)
uv run pyright
uv run mypy src/

# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/test_security.py -v

# Run tests with coverage
uv run pytest --cov=src/davinci_mcp --cov-report=html

# Security scanning
uv run bandit -r src/
uv run safety check

# Regenerate Doxygen documentation
doxygen Doxyfile
```

## Architecture

Data flow: **CLI -> MCP Server -> Resolve Client -> DaVinci Resolve scripting API**

```
src/davinci_mcp/
├── cli.py              # Entry point: prerequisite checks, colored output, click commands
├── server.py           # MCP protocol: routes tool calls and resource reads
├── resolve_client.py   # Wraps DaVinci Resolve Python API; owns connection lifecycle
├── types.py            # Runtime-checkable Protocol types (DaVinciProject, DaVinciTimeline, etc.)
├── tools/
│   └── __init__.py     # All 13 MCP tool definitions (name, description, inputSchema)
├── resources/
│   └── __init__.py     # All 7 MCP resource definitions (resolve://... URIs)
└── utils/
    ├── __init__.py
    └── platform.py     # Platform detection, PYTHONPATH setup, process checking
```

### Key Architectural Facts

- `server.py` implements `list_tools`, `call_tool`, `list_resources`,
  `read_resource` — all async. Tool dispatch and resource reads delegate
  to `resolve_client.py`.
- `resolve_client.py` uses lazy loading: current project fetched on demand
  and cached per-request. Raises from a custom exception hierarchy
  (`DaVinciResolveError` -> `DaVinciResolveNotRunningError`,
  `DaVinciResolveConnectionError`).
- `tools/__init__.py` and `resources/__init__.py` are pure data (lists of
  MCP tool/resource definitions). Adding a new tool: add definition in
  `tools/__init__.py`, add dispatch in `server.py`, implement in
  `resolve_client.py`.
- `types.py` uses `typing.Protocol` with `runtime_checkable` so Resolve
  objects can be type-checked without importing the Resolve module (which
  may not be present at type-check time).
- `utils/platform.py` handles OS differences: Windows uses `tasklist` for
  process detection and `ProgramData` paths; macOS/Linux use `pgrep` and
  standard POSIX paths.

### MCP Tool/Resource Inventory

**Tools (13):** `get_version`, `get_current_page`, `switch_page`,
`list_projects`, `get_current_project`, `open_project`, `create_project`,
`list_timelines`, `get_current_timeline`, `create_timeline`,
`switch_timeline`, `list_media_clips`, `import_media`.

**Resources (7):** `resolve://version`, `resolve://current-page`,
`resolve://projects`, `resolve://current-project`, `resolve://timelines`,
`resolve://current-timeline`, `resolve://media-clips`.

## Code Style and Standards

### Paradigm and Standards

Follow the **imperative paradigm** with close attention to procedural and
structured sub-paradigms. Project structure and formatting follow **GNU

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoyt-harness/davinci-mcp-professional](https://github.com/hoyt-harness/davinci-mcp-professional) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
