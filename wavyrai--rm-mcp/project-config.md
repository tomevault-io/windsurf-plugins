---
trigger: always_on
description: This is an MCP (Model Context Protocol) server that provides access to reMarkable tablet data. It's a Python project using FastMCP.
---

# Copilot Instructions for rm-mcp

## Project Overview

This is an MCP (Model Context Protocol) server that provides access to reMarkable tablet data. It's a Python project using FastMCP.

## Git Workflow

**Always work on feature branches and submit PRs. Never push directly to main.**

```bash
# Create a feature branch
git checkout -b feature/my-feature

# After making changes, push and create PR
git push origin feature/my-feature
# Then create PR via GitHub
```

Branch protection is enabled on `main` - all changes must go through pull requests with passing CI checks.

## Package Management

**Always use `uv` for all package management operations.**

```bash
# Install dependencies
uv sync

# Install with dev dependencies
uv sync --extra dev

# Add a new dependency
uv add <package>

# Add a dev dependency
uv add --dev <package>
```

**Never use:**
- `pip install` directly
- `pip freeze`
- `poetry`

## Running Tests

**Always run tests with:**

```bash
uv run pytest test_server.py -v
```

For specific test patterns:
```bash
# Run specific test class
uv run pytest test_server.py -v -k "TestClassName"

# Run with coverage
uv run pytest test_server.py -v --cov=server

# Skip slow tests if any
uv run pytest test_server.py -v -m "not slow"
```

**Important:** Tests use `pytest-asyncio` for async testing. All async tests should use the `@pytest.mark.asyncio` decorator.

## Building & Running

```bash
# Run the MCP server directly
uv run python server.py

# Interactive setup (one-time)
uv run python server.py --setup

# Run as installed package
uv run rm-mcp
```

## Code Quality

**Before committing, always run:**

```bash
# 1. Lint code (REQUIRED - CI will fail without this)
uv run ruff check .

# 2. Check formatting (REQUIRED - CI will fail without this)
uv run ruff format --check .

# 3. Run tests (REQUIRED - CI will fail without this)
uv run pytest test_server.py -v
```

**Fix issues automatically:**
```bash
# Fix lint issues
uv run ruff check . --fix

# Fix formatting
uv run ruff format .
```

**All three checks must pass before any commit or PR. CI runs on all PRs and must pass before merging.**

## Project Structure

```
rm-mcp/
├── server.py              # Entry point (backwards compatible)
├── rm_mcp/                # Main package
│   ├── __init__.py
│   ├── server.py          # FastMCP server initialization
│   ├── cli.py             # CLI (--setup, --register, server mode)
│   ├── api.py             # reMarkable Cloud API helpers
│   ├── cache.py           # Collection + extraction caching (L1/L2)
│   ├── index.py           # SQLite FTS5 document index
│   ├── models.py          # Protocol types
│   ├── paths.py           # Path resolution and root filtering
│   ├── responses.py       # Response formatting
│   ├── resources.py       # MCP resources + background loader
│   ├── prompts.py         # MCP prompts
│   ├── clients/
│   │   └── cloud.py       # reMarkable Cloud API client
│   ├── extract/           # Text/image extraction
│   │   ├── notebook.py    # .rm file parsing
│   │   ├── pdf.py         # PDF text extraction
│   │   ├── epub.py        # EPUB text extraction
│   │   └── render.py      # Page rendering (PNG/SVG)
│   ├── ocr/
│   │   └── sampling.py    # MCP sampling-based OCR
│   └── tools/             # MCP tool implementations
│       ├── _helpers.py    # Shared helpers and re-exports
│       ├── browse.py
│       ├── read.py
│       ├── search.py
│       ├── recent.py
│       ├── image.py
│       └── status.py
├── test_server.py         # Test suite
├── pyproject.toml         # Project config and dependencies
├── README.md              # User documentation (KEEP UPDATED)
└── .github/
    ├── copilot-instructions.md  # This file
    └── workflows/
        └── publish.yml    # PyPI + MCP Registry publishing
```

## Documentation Requirements

**README.md must always be kept in sync with the code:**

1. **Available Tools** - Update the tools table when adding/removing/renaming tools
2. **Example Usage** - Ensure examples work with current API
3. **Features** - Update feature list when capabilities change
4. **Installation** - Keep installation instructions accurate
5. **Dependencies** - Note any new required system dependencies (e.g., Tesseract for OCR)

When modifying `server.py`:
- If you add a new tool → Update README tools table and examples
- If you change tool parameters → Update README examples
- If you add new dependencies → Update README installation section

## MCP Tool Design Principles

When creating or modifying tools, follow these principles:

1. **Intent-based design** - Tools should map to user intents, not API endpoints
2. **XML-structured docstrings** - Use `<usecase>`, `<instructions>`, `<parameters>`, `<examples>` tags
3. **Response hints** - Always include `_hint` field suggesting next actions
4. **Educational errors** - Errors should explain what went wrong and how to fix it
5. **Minimal tool count** - Prefer fewer, more capable tools over many simple ones

Example tool structure:
```python
@mcp.tool()
def remarkable_example(param: str) -> str:
    """
    <usecase>Brief description of when to use this tool.</usecase>
    <instructions>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wavyrai/rm-mcp](https://github.com/wavyrai/rm-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
