---
trigger: always_on
description: An MCP (Model Context Protocol) server that provides tools for controlling Android devices using uiautomator2.
---

# uiautomator2-mcp-server

An MCP (Model Context Protocol) server that provides tools for controlling Android devices using uiautomator2.

## Project Overview

This project exposes Android device automation capabilities as MCP tools. It uses:
- **fastmcp** for MCP server implementation
- **cyclopts** for CLI argument parsing (via fastmcp dependency)
- **uiautomator2** for Android device control
- **anyio** for async operations
- **Pillow** for image handling

## Project Structure

```
src/u2mcp/
├── __init__.py          # Package init, exports version info
├── __main__.py          # Entry point for CLI commands
├── version.py           # Version info module
├── mcp.py               # MCP server factory and configuration
├── background.py        # Background task management
├── health.py            # ADB availability check and doctor command
├── helpers.py           # Helper functions for CLI output (tags, tools, info)
├── middlewares.py       # MCP middlewares for request/response handling
└── tools/
    ├── __init__.py      # Tools registry
    ├── action.py        # Touch/gesture tools (click, swipe, drag, etc.)
    ├── app.py           # App management tools
    ├── clipboard.py     # Clipboard read/write tools
    ├── delay.py         # Delay utility
    ├── device.py        # Device management tools
    ├── gesture.py       # Edge swipe and multitouch gestures
    ├── input.py         # Text input and keyboard tools
    ├── scrcpy.py        # Screen mirroring (scrcpy integration)
    ├── screenrecord.py  # Screen recording tools
    ├── system.py        # System controls (orientation, notification, unlock)
    ├── toast.py         # Toast message utilities
    └── xpath.py         # XPath-based element tools (primary)

.skills/                  # AI-driven testing skills
├── android-ui-test/
│   ├── SKILL.md         # Skill definition
│   ├── README.md        # Skill documentation
│   ├── examples/        # Usage examples
│   └── references/      # Technical specifications

tests/
├── conftest.py          # Pytest configuration and fixtures
├── unit/                # Fast unit tests
└── integration/         # Integration tests requiring real devices
```

## Development Setup

```bash
# Install dependencies (uses uv lock file)
uv sync --dev
```

Activate virtual environment

```bash
source .venv/bin/activate  # Linux/macOS
```

or Windows:

```powershell
.venv\Scripts\activate
```

## Running Tests

```bash
# Run all tests
pytest

# Run only unit tests (fast, no device required)
pytest -m unit

# Run integration tests (requires Android device)
pytest -m integration

# Run with coverage
pytest --cov=src/u2mcp --cov-report=html
```

## Test Markers

- `unit` - Unit tests (fast, no external dependencies)
- `integration` - Integration tests (may require external services)
- `slow` - Slow running tests
- `device` - Tests that require actual Android device

## Code Style

- Python >= 3.11
- Type hints required for all function signatures
- Use `from __future__ import annotations` for deferred evaluation
- Use `typing_extensions` for Python < 3.12 compatibility
- Use `Annotated[type, Parameter(...)]` for CLI parameters with custom names
- Ruff for linting
- mypy for type checking with `types-lxml` and `types-retry` for proper type coverage

## CLI Entry Points

The server can be invoked using any of these commands (they are aliases):
- `u2mcp` - Short form (primary)
- `uiautomator2-mcp` - Full form
- `uiautomator2-mcp-server` - Descriptive form

## MCP Tools

All tools are decorated with `@mcp.tool()` and accept a `serial` parameter to identify the target device.

Tools are organized into modules:
- **action.py** - Coordinate-based touch/gestures (click, swipe, drag)
- **xpath.py** - XPath-based element location and interaction (primary, LLM-optimized)
- **device.py** - Device management and screen capture
- **app.py** - Application lifecycle and management
- **input.py** - Text input and keyboard control
- **gesture.py** - Edge swipes and multitouch gestures
- **system.py** - System controls (orientation, notification, unlock)
- **screenrecord.py** - Screen recording
- **scrcpy.py** - Screen mirroring
- **clipboard.py** - Clipboard read/write
- **delay.py** - Delay utility
- **toast.py** - Toast messages

Use `u2mcp tools` to list all available tools.

## Tool Function Design Principles

**Core Philosophy: Naming, types, and docstrings are for LLM consumption.**

The audience for tool function signatures and documentation is the LLM, not human developers. Design for LLM understanding.

### 1. Mandatory Type Annotations

All parameters and return values MUST have explicit type annotations.

### 2. Minimize Type Complexity

Keep parameter and return types simple for LLM understanding.

**Avoid:**
- `Optional[T]` - Use separate required parameters or provide defaults
- `Dict[str, Any]` - Use specific dataclasses or named tuples
- `List[T]` - Use fixed-size tuples or separate parameters
- `Union[T1, T2]` - Use separate functions or overload

**Prefer:**
- `str`, `int`, `float`, `bool`
- `tuple[int, int, int, int]` for coordinates
- Literal types like `Literal["left", "right", "up", "down"]`

### 3. No Redundant Return Type Annotations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanbro/uiautomator2-mcp-server](https://github.com/tanbro/uiautomator2-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
