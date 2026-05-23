---
trigger: always_on
description: A read-only [Model Context Protocol](https://modelcontextprotocol.io/) server that enables LLMs to interact with NetBox infrastructure data. Built with FastMCP and designed for use by NetBox operators.
---

# NetBox MCP Server

## Core Concept

A read-only [Model Context Protocol](https://modelcontextprotocol.io/) server that enables LLMs to interact with NetBox infrastructure data. Built with FastMCP and designed for use by NetBox operators.

**Your role**: Help contributors design and implement features within the project's stated scope (see [CONTRIBUTING.md](CONTRIBUTING.md)). Challenge proposals that fall outside scope before implementation begins, not after. Ask clarifying questions and challenge assumptions when needed.

## Tech Stack

- **Python**: >=3.11, <3.15
- **Package Manager**: uv
- **MCP Framework**: FastMCP >=3.0.0
- **HTTP Client**: httpx
- **NetBox API**: REST API via token authentication

## Project Structure

```text
.
├── src/
│   └── netbox_mcp_server/
│       ├── __init__.py          # Package initialization with __version__
│       ├── __main__.py          # Entry point for module execution
│       ├── server.py            # Main MCP server with tool definitions
│       ├── netbox_client.py     # NetBox REST API client abstraction
│       ├── netbox_types.py      # NetBox object type mappings
│       └── config.py            # Settings and logging configuration
├── tests/                        # Test suite
├── .github/workflows/            # CI/CD automation
├── pyproject.toml               # Dependencies and project metadata
├── README.md                    # User-facing documentation
├── CHANGELOG.md                 # Auto-generated release notes
└── LICENSE                      # Apache 2.0 license
```

**Design Pattern**: Clean separation between MCP server logic (`server.py`) and NetBox API client (`netbox_client.py`) to support future plugin-based implementations.

## Common Commands

```bash
# Install dependencies (ONLY use uv, NEVER pip)
uv sync

# Run the server locally (requires env vars)
NETBOX_URL=https://netbox.example.com/ NETBOX_TOKEN=<token> uv run netbox-mcp-server

# Alternative: module execution
uv run -m netbox_mcp_server

# Add to Claude Code (for development/testing)
claude mcp add --transport stdio netbox \
  --env NETBOX_URL=https://netbox.example.com/ \
  --env NETBOX_TOKEN=<token> \
  -- uv --directory /path/to/netbox-mcp-server run netbox-mcp-server
```

## Development Philosophy

- **Simplicity over cleverness**: Write simple, straightforward code that's easy to understand
- **Readability first**: Code is read 10x more than it's written - optimize for the reader
- **Build iteratively**: Start with minimal functionality, verify it works, then add complexity
- **DRY (Don't Repeat Yourself)**: Extract common patterns, but only after the third occurrence
- **Early returns**: Use early returns to avoid nested conditions and improve readability
- **Descriptive names**: Use clear variable and function names that explain intent
- **Less code = less debt**: Minimize code footprint; the best code is no code at all
- **Test frequently**: Test with realistic inputs and validate outputs as you build
- **Functional where clear**: Use functional, stateless approaches when they improve clarity
- **Clean core logic**: Keep business logic clean; push implementation details to the edges

## Version Management

This project uses [python-semantic-release](https://python-semantic-release.readthedocs.io/) for automated version management. Versions are automatically determined from commit messages following [Conventional Commits](https://www.conventionalcommits.org/).

**Release triggers:**

- `feat:` commits trigger minor version bumps (1.0.0 → 1.1.0)
- `fix:` and `perf:` commits trigger patch version bumps (1.0.0 → 1.0.1)
- Commits with `BREAKING CHANGE:` in the body trigger major version bumps (1.0.0 → 2.0.0)
- `docs:`, `test:`, `chore:`, `ci:`, `refactor:` commits are logged but don't trigger releases

**Workflow:**

- Merge to `main` automatically triggers release analysis
- If commits warrant a release, version is bumped and CHANGELOG updated
- GitHub Release is created with auto-generated release notes

## Code Standards

### Python Conventions

- **Type hints required**: All function parameters and return types must be annotated
- **Docstrings**: Use Google-style docstrings for all public functions and classes
- **Line length**: 88 characters maximum (Ruff/Black standard)
- **Naming conventions**:
  - Functions and variables: `snake_case`
  - Classes: `PascalCase`
  - Constants: `UPPER_SNAKE_CASE`
- **String formatting**: Use f-strings for string formatting
- **Imports**: Absolute imports preferred, group standard library → third-party → local
- **Error handling**: Never use bare `except` - specify exception types (e.g., `except ValueError:`); raise descriptive exceptions and let FastMCP handle error responses

### Code Style

```python
# ✅ Good: Clear types, descriptive names, proper error handling, comprehensive docstrings
def netbox_get_objects(object_type: str, filters: dict) -> list[dict]:
    """Get objects from NetBox based on their type and filters.

    Args:
        object_type: String representing the NetBox object type (e.g. "devices")
        filters: Dictionary of filters to apply to the API call

    Returns:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [netboxlabs/netbox-mcp-server](https://github.com/netboxlabs/netbox-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
