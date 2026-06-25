---
trigger: always_on
description: This file provides guidance to AI agents (including Claude Code, Cursor, and other LLM-powered tools) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents (including Claude Code, Cursor, and other LLM-powered tools) when working with code in this repository.

## CRITICAL REQUIREMENTS

### Test Success
- ALL tests MUST pass for code to be considered complete and working
- Never describe code as "working as expected" if there are ANY failing tests
- Even if specific feature tests pass, failing tests elsewhere indicate broken functionality
- Changes that break existing tests must be fixed before considering implementation complete
- A successful implementation must pass linting, type checking, AND all existing tests

## Project Overview

libtmux-mcp is an MCP (Model Context Protocol) server for tmux, powered by [libtmux](https://github.com/tmux-python/libtmux). It gives AI agents (Claude Code, Claude Desktop, Codex CLI, Gemini CLI, Cursor) programmatic control over tmux sessions.

Key features:
- MCP tools across 6 modules: server, session, window, pane, options, environment
- `tmux://` URI resources for browsing tmux hierarchy
- Safety tier middleware (readonly, mutating, destructive)
- Socket isolation for multi-server safety
- Pydantic models for all tool outputs
- Full type safety (mypy strict)

The core tmux ORM is provided by [libtmux](https://libtmux.git-pull.com/) - this package wraps it as an MCP server.

## Development Environment

This project uses:
- Python 3.10+
- [uv](https://github.com/astral-sh/uv) for dependency management
- [ruff](https://github.com/astral-sh/ruff) for linting and formatting
- [mypy](https://github.com/python/mypy) for type checking
- [pytest](https://docs.pytest.org/) for testing
  - [pytest-watcher](https://github.com/olzhasar/pytest-watcher) for continuous testing

## Common Commands

### Setting Up Environment

```bash
# Install dependencies
uv pip install --editable .
uv pip sync

# Install with development dependencies
uv pip install --editable . -G dev
```

### Running Tests

```bash
# Run all tests
just test
# or directly with pytest
uv run pytest

# Run a single test file
uv run pytest tests/test_pane_tools.py

# Run a specific test
uv run pytest tests/test_pane_tools.py::test_send_keys

# Run tests with test watcher
just start
# or
uv run ptw .

# Run tests with doctests
uv run ptw . --now --doctest-modules
```

### Linting and Type Checking

```bash
# Run ruff for linting
just ruff
# or directly
uv run ruff check .

# Format code with ruff
just ruff-format
# or directly
uv run ruff format .

# Run ruff linting with auto-fixes
uv run ruff check . --fix --show-fixes

# Run mypy for type checking
just mypy
# or directly
uv run mypy src tests

# Watch mode for linting (using entr)
just watch-ruff
just watch-mypy
```

### Development Workflow

Follow this workflow for code changes:

1. **Format First**: `uv run ruff format .`
2. **Run Tests**: `uv run pytest`
3. **Run Linting**: `uv run ruff check . --fix --show-fixes`
4. **Check Types**: `uv run mypy`
5. **Verify Tests Again**: `uv run pytest`

### Documentation

```bash
# Build documentation
just build-docs

# Start documentation server with auto-reload
just start-docs

# Update documentation CSS/JS
just design-docs
```

## Code Architecture

libtmux-mcp wraps libtmux's tmux hierarchy as MCP tools and resources:

```
tmux hierarchy: Server > Session > Window > Pane
```

### Core Modules

1. **Entry Point** (`src/libtmux_mcp/__init__.py`)
   - `main()` function, console script entry point
   - Guards against missing fastmcp dependency

2. **Server** (`src/libtmux_mcp/server.py`)
   - Creates and configures the FastMCP instance
   - Builds server instructions with agent context
   - Safety tier validation from `LIBTMUX_SAFETY` env var

3. **Utils** (`src/libtmux_mcp/_utils.py`)
   - Thread-safe server caching by (socket_name, socket_path, tmux_bin) tuple
   - Object resolvers: `_resolve_session()`, `_resolve_window()`, `_resolve_pane()`
   - Serializers: `_serialize_session()`, `_serialize_window()`, `_serialize_pane()`
   - QueryList filter application with validation
   - `handle_tool_errors` decorator for standardized error handling
   - Safety tier tags and annotation presets

4. **Models** (`src/libtmux_mcp/models.py`)
   - Pydantic models for all tool outputs
   - `SessionInfo`, `WindowInfo`, `PaneInfo`, `PaneContentMatch`
   - `ServerInfo`, `OptionResult`, `EnvironmentResult`, `WaitForTextResult`

5. **Middleware** (`src/libtmux_mcp/middleware.py`)
   - `SafetyMiddleware` gates tools by tier (readonly/mutating/destructive)
   - Fail-closed: tools without a recognized tier tag are denied

6. **Tools** (`src/libtmux_mcp/tools/`)
   - `server_tools.py` - list_sessions, create_session, kill_server, get_server_info
   - `session_tools.py` - list_windows, create_window, rename_session, kill_session
   - `window_tools.py` - list_panes, split_window, rename_window, kill_window, select_layout, resize_window
   - `pane_tools.py` - send_keys, capture_pane, resize_pane, kill_pane, set_pane_title, get_pane_info, clear_pane, search_panes, wait_for_text
   - `option_tools.py` - show_option, set_option
   - `env_tools.py` - show_environment, set_environment

7. **Resources** (`src/libtmux_mcp/resources/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tmux-python/libtmux-mcp](https://github.com/tmux-python/libtmux-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
