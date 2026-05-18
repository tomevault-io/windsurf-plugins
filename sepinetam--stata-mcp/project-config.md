---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Stata-MCP is an MCP (Model Context Protocol) server that enables LLMs to execute Stata commands and perform statistical/regression analysis. It supports:
- **MCP server mode**: FastMCP-based server exposing Stata tools to LLMs
- **Agent mode**: Interactive Stata analysis via OpenAI Agents SDK
- **CLI tools**: Direct command-line access to all Stata capabilities

Current version: **1.16.2** | License: **AGPL-3.0** | Python: **>=3.11**

## Common Development Commands

### Environment Setup
```bash
# Install dependencies and create virtual environment
uv sync

# Install the package in development mode
uv pip install -e .

# Verify installation
stata-mcp --version

# Run diagnostics to check system health
stata-mcp doctor

# NOTE: --usable is deprecated since v1.14.3, use "stata-mcp doctor" instead
```

### Running Tests
```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_server_parser.py
pytest tests/test_server_registration.py
```

### Building and Distribution
```bash
# Build source distribution and wheels
uv build

# Build specific formats
uv build --sdist    # Source distribution only
uv build --wheel    # Wheel only

# Specify output directory
uv build --out-dir dist/
```

### Running the Application

#### MCP Server Mode (default)
```bash
# Start MCP server with stdio transport (default)
stata-mcp

# Start with specific transport
stata-mcp -t http    # HTTP transport
stata-mcp -t sse     # SSE transport

# Start with tool profile selection
stata-mcp server                     # All tools, stdio (same as bare command)
stata-mcp server --core              # Core tools only (stata_do, get_data_info, help)
stata-mcp server --all -t http       # All tools, HTTP transport
stata-mcp server --core -t http      # Core tools, HTTP transport
```

#### Agent Mode (Deprecated)
> Agent mode is deprecated and will be removed in a future version. Use MCP server mode instead.

```bash
# Run interactive agent mode (deprecated)
stata-mcp agent run

# Or use uvx for direct execution (deprecated)
uvx stata-mcp agent run
```

#### Utility Commands
```bash
# Run diagnostics to check system health (replaces deprecated --usable)
stata-mcp doctor
stata-mcp doctor --verbose          # Detailed output
stata-mcp doctor --json             # JSON output
stata-mcp doctor --check stata      # Run specific check(s)

# Update stata-mcp to latest version
stata-mcp update
stata-mcp update --check            # Check if update is available
stata-mcp update --dry-run          # Show detected method without updating
stata-mcp update --method pip       # Force specific update method (auto/pip/uv-tool/homebrew)

# Manage configuration
stata-mcp config                    # Show current config
stata-mcp config cli set            # Auto-detect and set STATA_CLI path
stata-mcp config cli set /path/to/stata  # Set specific STATA_CLI path

# Run local Stata tools via CLI
stata-mcp tool ado-install <package> [--source ssc|net|github]
stata-mcp tool do <dofile_path> [--log-file-name NAME]
stata-mcp tool help <command>
stata-mcp tool data-info <data_path> [--vars-list var1 var2]
stata-mcp tool read-log <file_path> [--output-format full|core|dict]

# Install to MCP clients
stata-mcp install                   # Default: Claude Desktop
stata-mcp install -c cc             # Claude Code
stata-mcp install -c gemini         # Gemini CLI
stata-mcp install -c cursor         # Cursor
stata-mcp install -c cline          # Cline
stata-mcp install -c codex          # Codex
stata-mcp install -c opencode       # OpenCode
stata-mcp install -c openclaw       # OpenClaw
stata-mcp install --all             # Install to all supported clients
stata-mcp install --json-file PATH  # Install to custom config file

# Docker-based sandbox installation
stata-mcp sandbox-install -l /path/to/stata.lic
stata-mcp sandbox-install -l /path/to/stata.lic -c cursor --cpus 2 --memory 4g

# Check version
stata-mcp --version
```

### Development with uvx
```bash
# Run without local installation
uvx stata-mcp --version
uvx stata-mcp agent run
uvx stata-mcp doctor
```

## Source Layout

```
src/stata_mcp/
├── __init__.py              # Lazy-loading exports: stata_mcp (server), main (CLI)
├── mcp_servers.py           # FastMCP server with _TOOL_REGISTRY and register_tools()
├── config.py                # Unified config management (TOML + env vars)
├── api/                     # Tool API wrappers (thin layer over core logic)
│   ├── _runtime.py          # RuntimeContext dataclass for execution contexts
│   ├── stata_do.py          # stata_do() with security guard + RAM monitor
│   ├── get_data_info.py     # Data file analysis dispatcher
│   ├── read_log.py          # Log file reader (text and SMCL formats)
│   ├── stata_help.py        # Stata command documentation
│   ├── ado_install.py       # Package installer (SSC/GitHub/net)
│   └── write_dofile.py      # Do-file creator (deprecated)
├── cli/                     # Command-line interface
│   ├── _cli.py              # Entry point and subcommand routing
│   ├── _parsers.py          # Argument parser definitions for all subcommands

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SepineTam/stata-mcp](https://github.com/SepineTam/stata-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
