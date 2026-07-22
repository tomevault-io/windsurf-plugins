---
trigger: always_on
description: provides. Its ready-to-use tmux hierarchy object fixtures are:
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

libtmux is a typed Python library that provides an Object-Relational Mapping (ORM) wrapper for interacting programmatically with [tmux](https://github.com/tmux/tmux), a terminal multiplexer.

Key features:
- Manage tmux servers, sessions, windows, and panes programmatically
- Typed Python API with full type hints
- Built on tmux's target and formats system
- Powers [tmuxp](https://github.com/tmux-python/tmuxp), a tmux workspace manager
- Provides pytest fixtures for testing with tmux

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
uv run pytest tests/test_pane.py

# Run a specific test
uv run pytest tests/test_pane.py::test_send_keys

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

libtmux follows an object-oriented design that mirrors tmux's hierarchy:

```
Server (tmux server instance)
  └─ Session (tmux session)
      └─ Window (tmux window)
          └─ Pane (tmux pane)
```

### Core Modules

1. **Server** (`src/libtmux/server.py`)
   - Represents a tmux server instance
   - Manages sessions
   - Executes tmux commands via `tmux()` method
   - Entry point for most libtmux interactions

2. **Session** (`src/libtmux/session.py`)
   - Represents a tmux session
   - Manages windows within the session
   - Provides session-level operations (attach, kill, rename, etc.)

3. **Window** (`src/libtmux/window.py`)
   - Represents a tmux window
   - Manages panes within the window
   - Provides window-level operations (split, rename, move, etc.)

4. **Pane** (`src/libtmux/pane.py`)
   - Represents a tmux pane (terminal instance)
   - Provides pane-level operations (send-keys, capture, resize, etc.)
   - Core unit for command execution and output capture

5. **Common** (`src/libtmux/common.py`)
   - Base classes and shared functionality
   - `TmuxRelationalObject` and `TmuxMappingObject` base classes
   - Format handling and command execution

6. **Formats** (`src/libtmux/formats.py`)
   - Tmux format string constants
   - Used for querying tmux state

7. **Neo** (`src/libtmux/neo.py`)
   - Modern query interface and dataclass-based objects
   - Alternative to traditional ORM-style objects

8. **pytest Plugin** (`src/libtmux/pytest_plugin.py`)
   - Provides fixtures for testing with tmux
   - Creates temporary tmux servers and sessions

## Testing Strategy

libtmux uses pytest for testing with custom fixtures. The pytest plugin
(`src/libtmux/pytest_plugin.py`) is the source of truth for the fixtures it
provides. Its ready-to-use tmux hierarchy object fixtures are:

- `server`: A tmux server instance for testing
- `session`: A tmux session for testing

These fixtures handle setup and teardown automatically, creating isolated test
environments. For regular tests, derive windows and panes from the `session`
fixture with `session.new_window(...)` and `window.active_pane`. The `window`
and `pane` names described below are doctest namespace values, not pytest
fixtures.

### Testing Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tmux-python/libtmux](https://github.com/tmux-python/libtmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
