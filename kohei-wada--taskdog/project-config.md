---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Taskdog is a task management system built with Python. It provides a CLI/TUI interface and a REST API server for managing tasks with time tracking, dependencies, schedule optimization, and rich terminal output. The codebase follows Clean Architecture principles and is organized as a UV workspace monorepo.

### Monorepo Structure

The repository uses UV workspace with five packages:

**taskdog-core** (`packages/taskdog-core/`): Core business logic and infrastructure

- Domain entities, use cases, repositories, validators
- SQLite + SQLAlchemy ORM with Alembic migrations
- Schedule optimization algorithms (9 strategies)
- No UI dependencies - pure business logic

**taskdog-server** (`packages/taskdog-server/`): FastAPI REST API server

- Depends on: taskdog-core
- FastAPI application with automatic OpenAPI docs
- API routers: tasks, lifecycle, relationships, analytics, notes, audit, websocket
- Pydantic models for request/response validation
- Health check and comprehensive error handling

**taskdog-client** (`packages/taskdog-client/`): HTTP client for server communication

- Depends on: taskdog-core
- TaskdogClient class for API communication
- Used by CLI/TUI to communicate with the server

**taskdog-ui** (`packages/taskdog-ui/`): CLI and TUI interfaces

- Depends on: taskdog-core, taskdog-client
- Click-based CLI commands with Rich output
- Textual-based full-screen TUI
- Renderers for tables, Gantt charts, and markdown notes

**taskdog-mcp** (`packages/taskdog-mcp/`): MCP server for Claude Desktop integration

- Depends on: taskdog-client
- Model Context Protocol server implementation
- Enables Claude Desktop to manage tasks via natural language

### Data Storage & Configuration

**Tasks**: Stored in SQLite database `tasks.db` at `$XDG_DATA_HOME/taskdog/tasks.db` (fallback: `~/.local/share/taskdog/tasks.db`)

**Config**: Optional TOML files at `$XDG_CONFIG_HOME/taskdog/` (fallback: `~/.config/taskdog/`)

- **core.toml** (business logic):
  - Sections: `[time]`, `[region]`, `[storage]`
  - Settings: default_start_time, default_end_time, country, database_url, backend
- **server.toml** (server-specific):
  - Sections: `[auth]`
  - Settings: enabled, api_keys
- **cli.toml** (CLI/TUI infrastructure):
  - Sections: `[api]`, `[ui]`, `[notes]`, `[keybindings]`
  - Settings: host, port, api_key, theme
- **mcp.toml** (MCP server settings):
  - Sections: `[api]`
  - Settings: host, port, api_key
- Priority: Environment vars > CLI args > Config file > Defaults
- Server host/port: CLI arguments (`taskdog-server --host --port`)
- CLI/TUI connection: `cli.toml` or `TASKDOG_API_HOST`/`TASKDOG_API_PORT` env vars
- Access via `ctx.obj.config` (CLI) or `context.config` (TUI)

## Development Commands

```bash
# Setup (from workspace root)
make install-dev                 # Install all packages with dev dependencies (editable mode)
make install-local               # Install all packages locally for development (alias for install-core install-server install-ui)
make install                     # Install taskdog and taskdog-server as global commands via uv tool

# Per-package installation (for development)
make install-core                # Install taskdog-core only
make install-server              # Install taskdog-server (includes core)
make install-ui                  # Install taskdog-ui (includes core)

# Testing (all tests include coverage report)
make test                        # Run all tests with coverage (core + server + ui)
make test-core                   # Run taskdog-core tests with coverage
make test-server                 # Run taskdog-server tests with coverage
make test-ui                     # Run taskdog-ui tests with coverage

# Single test file (run from package directory)
cd packages/taskdog-core && PYTHONPATH=src uv run python -m pytest tests/test_module.py -v
# Specific test method
cd packages/taskdog-core && PYTHONPATH=src uv run python -m pytest tests/test_module.py::TestClass::test_method -v

# Code Quality
make lint                        # Ruff linter on all packages
make format                      # Ruff formatter + auto-fix on all packages
make typecheck                   # mypy on all packages (progressive type checking, Phase 4)
make check                       # lint + typecheck

# Cleanup
make clean                       # Clean build artifacts and cache (stops systemd service)
make uninstall                   # Uninstall all global commands (removes systemd service)
make reinstall                   # Clean + reinstall all global commands

# Running the applications
taskdog --help                   # CLI/TUI (after make install)
taskdog-server --help            # API server (after make install)

# Development mode (without installation)
cd packages/taskdog-ui && PYTHONPATH=src uv run python -m taskdog.cli_main --help
cd packages/taskdog-server && PYTHONPATH=src uv run python -m taskdog_server.main --help

# Server examples
taskdog-server                   # Start on default 127.0.0.1:8000
taskdog-server --host 0.0.0.0 --port 3000  # Custom host/port

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kohei-Wada/taskdog](https://github.com/Kohei-Wada/taskdog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
