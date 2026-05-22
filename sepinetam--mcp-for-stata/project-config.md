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

License: **AGPL-3.0** | Python: **>=3.11**

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

#### Utility Commands
```bash
# Run diagnostics to check system health (replaces deprecated --usable)
stata-mcp doctor
stata-mcp doctor --verbose          # Detailed output
stata-mcp doctor --json             # JSON output
stata-mcp doctor --check stata      # Run specific check(s)

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
│   └── _handlers.py         # Command handler implementations
├── stata/                   # Stata integration layer
│   ├── stata_finder/        # Platform-specific Stata executable locator
│   │   ├── finder.py        # Factory dispatcher
│   │   ├── base.py          # Abstract base
│   │   ├── macos.py         # macOS implementation
│   │   ├── windows.py       # Windows implementation
│   │   └── linux.py         # Linux implementation
│   ├── stata_controller/    # Interactive pexpect-based Stata session
│   ├── stata_do/            # Batch do-file execution with subprocess
│   └── builtin_tools/
│       ├── ado_install/     # SSC_Install, NET_Install, GITHUB_Install
│       ├── help/            # StataHelp with disk caching
│       └── stata_log/       # Log readers: text and SMCL formats
├── data_info/               # Data file analysis handlers
│   ├── base.py              # DataInfoBase ABC, Series dataclasses
│   ├── csv.py               # CSV handler
│   ├── dta.py               # Stata .dta handler
│   ├── xlsx.py              # Excel handler
│   └── spss.py              # SPSS .sav handler
├── guard/                   # Security validation
│   ├── validator.py         # GuardValidator, RiskItem, SecurityReport
│   └── blacklist.py         # DANGEROUS_COMMANDS, DANGEROUS_PATTERNS
├── monitor/                 # Process monitoring
│   ├── base.py              # MonitorBase ABC
│   └── ram_monitor.py       # RAMMonitor (threading + psutil)
├── agent_as/                # Agent-mode components
│   ├── agent_base.py        # AgentBase ABC (OpenAI Agents SDK)
│   ├── repl_agents.py       # REPLAgent with SQLiteSession + MCP server
│   ├── set_model.py         # Model configuration helpers
│   ├── agent_as_tool/       # stata_agent, adversarial_thinking_agent, any_as_tools

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SepineTam/mcp-for-stata](https://github.com/SepineTam/mcp-for-stata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
