---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Codex Bridge** is a lightweight MCP (Model Context Protocol) server that enables Claude Code to interact with OpenAI's Codex AI through the official CLI. The project follows extreme simplicity principles from Carmack and Torvalds - doing ONE thing well: bridging Claude to Codex CLI.

**Key Characteristics:**
- Non-interactive execution with structured output formats (text, JSON, code)
- Stateless architecture with no session management
- Direct subprocess integration for optimal performance
- Batch processing support for CI/CD workflows
- Pipeline-friendly stdin support

## Development Commands

### Prerequisites
- **Codex CLI**: Install from OpenAI (official CLI)
- **Authentication**: Setup OpenAI API credentials
- **Verify**: `codex --version`

### Installation & Setup

**Development Mode:**
```bash
# Clone and install in development mode
git clone https://github.com/shelakh/codex-bridge.git
cd codex-bridge
pip install -e .

# Run directly from source
python -m src
```

**Production Installation:**
```bash
# Install from PyPI
pip install codex-bridge

# Or use uvx (recommended)
uvx codex-bridge
```

**Claude Code Integration:**
```bash
# Production installation (recommended)
claude mcp add codex-bridge -s user -- uvx codex-bridge

# Development mode (from local source)
claude mcp add codex-bridge -s user -- python -m src
```

### Testing & Verification
```bash
# Test CLI availability
codex --version

# Test basic functionality
python -c "from src.mcp_server import execute_codex_prompt; print('MCP server loaded successfully')"

# Test package installation
python -c "import src; print(f'Codex Bridge v{src.__version__}')"
```

### Build & Distribution
```bash
# Clean build
rm -rf dist/ build/ *.egg-info

# Build package
uvx --from build pyproject-build

# Verify build
pip install dist/*.whl
python -c "import src; print('Package works!')"
```

## Architecture

### Core Design Principles
- **CLI-First**: Direct subprocess calls to `codex` command
- **Stateless**: Each tool call is independent with no session state
- **Non-Interactive**: Structured execution with JSON/text/code formats
- **Configurable Timeout**: Default 90-second execution time (configurable via CODEX_TIMEOUT)
- **Git Repository Check**: Configurable via CODEX_SKIP_GIT_CHECK for non-git directories
- **Fail-Fast**: Clear error messages with simple error handling
- **Zero Dependencies**: Only `mcp>=1.0.0` and external Codex CLI

### Key Components

**`src/mcp_server.py`** - Main server implementation
- `consult_codex(query, directory, format, timeout)` - Non-interactive consultation
- `consult_codex_with_stdin(stdin_content, prompt, directory, format, timeout)` - Pipeline consultation
- `consult_codex_batch(queries, directory, format)` - Batch processing
- `_format_response()` - Output formatting for text/JSON/code

**Codex CLI Integration:**
- Uses the official OpenAI Codex CLI without model selection
- Single unified interface for AI-powered code assistance

### File Structure
```
codex-bridge/
├── src/
│   ├── __init__.py              # Package entry point and version
│   ├── __main__.py              # Module execution entry point  
│   └── mcp_server.py            # Main MCP server implementation
├── .github/                     # GitHub templates and workflows
├── pyproject.toml               # Python package configuration
├── README.md                    # Main documentation
├── CONTRIBUTING.md              # Development guidelines
├── SECURITY.md                  # Security policy
├── CHANGELOG.md                 # Release history
└── LICENSE                      # MIT license
```

## MCP Tools Available

### `consult_codex`
- **Purpose**: Non-interactive Codex consultation with structured output
- **Parameters**: 
  - `query` (required): The prompt to send to Codex
  - `directory` (required): Working directory for the query
  - `format` (optional): Output format - "text", "json", or "code" (default: "text")
  - `timeout` (optional): Timeout in seconds (overrides env var)
- **Use Case**: Direct code generation, analysis, structured responses
- **Example**: Code completion, refactoring suggestions

### `consult_codex_with_stdin` 
- **Purpose**: Pipeline-friendly consultation with stdin content
- **Parameters**: 
  - `stdin_content` (required): Content to pipe (file contents, diffs, logs)
  - `prompt` (required): The prompt to process the stdin content
  - `directory` (required): Working directory for the query  
  - `format` (optional): Output format - "text", "json", or "code" (default: "text")
  - `timeout` (optional): Timeout in seconds (overrides env var)
- **Use Case**: CI/CD workflows, processing file contents, diff analysis
- **Example**: Code reviews, log analysis, file processing

### `consult_codex_batch`
- **Purpose**: Batch processing for multiple queries - CI/CD automation
- **Parameters**: 
  - `queries` (required): List of query dictionaries with 'query' and optional 'timeout'
  - `directory` (required): Working directory for all queries
  - `format` (optional): Output format - currently only "json" supported

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eLyiN/codex-bridge](https://github.com/eLyiN/codex-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
