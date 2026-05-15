---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Git MCP Server is a Model Context Protocol (MCP) server that enables issue-to-code automation for Claude Code, Gemini CLI, and Codex. It provides unified Git platform access for GitLab and GitHub (including GitHub Enterprise) through MCP tools and slash commands for complete development workflows.

## Key Commands

### Development & Testing
```bash
# Install from source (development)
uv sync --all-extras
uv run git-mcp-server

# Install globally for testing
uv tool install --from . git_mcp_server --force

# Run linting and security checks
uv run ruff check git_mcp/
uv run ruff format git_mcp/
uv run bandit -r git_mcp/
uv run mypy git_mcp/ --ignore-missing-imports --no-strict-optional

# Run pre-commit hooks manually (includes ruff, bandit, mypy, yaml/json checks)
uv run pre-commit run --all-files

# Run tests
uv run pytest

# Run tests with coverage
uv run pytest --cov=git_mcp --cov-report=term-missing

# Run specific test file
uv run pytest tests/test_logging.py

# Test CLI entry points
uv run git-mcp --help
uv run git-mcp-server --help

# Test MCP server directly
echo '{"jsonrpc": "2.0", "method": "initialize", "params": {"protocolVersion": "2024-11-05", "capabilities": {}, "clientInfo": {"name": "test", "version": "1.0.0"}}, "id": 1}' | git-mcp-server

# Build distribution packages
uv build

# Run development version in interactive MCP mode
uv run mcp dev git_mcp/mcp_server.py
```

### Installation & Setup
```bash
# Install from PyPI and configure Claude Code integration
uv tool install git_mcp_server
git-mcp-server --install-claude

# Install from PyPI and configure Gemini CLI integration
git-mcp-server --install-gemini

# Install from PyPI and configure Codex integration
git-mcp-server --install-codex

# Local development installation
./install.sh
```

### Configuration Management
```bash
# Add platform configuration
git-mcp config add my-gitlab gitlab --url https://gitlab.com
git-mcp config add my-github github --url https://github.com

# Test platform connections
git-mcp config test my-gitlab
git-mcp config test my-github

# List configured platforms
git-mcp config list

# Refresh username from token
git-mcp config refresh-username my-gitlab
```

## Architecture

### Core Components

**MCP Server** (`git_mcp/mcp_server.py`):
- Main MCP interface with ~25 tools for platform management, issues, projects, and merge requests
- Supports automatic Claude Code and Gemini CLI integration with `--install-claude` and `--install-gemini` flags
- Resources for platform configuration and project data

**CLI Interface** (`git_mcp/cli.py`):
- Click-based CLI with commands for config, project, issue, and MR management
- Global context management with configurable output formats (table, json, yaml)
- Async command execution with proper error handling

**Configuration System** (`git_mcp/core/config.py`):
- YAML-based config storage at `~/.git-mcp/config.yaml`
- Secure keyring-based token storage
- Automatic username fetching from platform APIs
- Platform, defaults, and aliases management

**Platform Adapters** (`git_mcp/platforms/`):
- Base adapter interface (`base.py`) with common Git platform operations
- GitLab implementation (`gitlab.py`) with full API coverage
- GitHub implementation (`github.py`) with full API coverage including GitHub Enterprise
- Extensible design for additional platforms

**Service Layer** (`git_mcp/services/platform_service.py`):
- Unified service interface abstracting platform-specific implementations
- Handles platform routing, error handling, and data normalization
- Async operations with proper exception management

### Slash Commands Integration

**Claude Code Commands** (`git_mcp/claude_commands/`):
- Complete issue-to-code workflow: `/issue` → `/plan` → `/implement` → `/test` → `/doc` → `/pr`
- Installed to `~/.claude/commands/` during setup
- Markdown-based command definitions with MCP tool integration

**Gemini CLI Commands** (`git_mcp/gemini_commands/`):
- TOML-based command definitions for Gemini CLI
- Installed to `~/.gemini/commands/` during setup
- Same workflow commands adapted for Gemini CLI format

**Codex Commands** (`git_mcp/codex_commands/`):
- Markdown-based command definitions for Codex integration
- Installed to `~/.codex/prompts/` during setup
- Same workflow commands adapted for Codex prompt format
- Includes memory integration via `~/.codex/AGENTS.md`

### Key Design Patterns

- **Async-first**: All platform operations are async for better performance
- **Service abstraction**: Platform operations go through unified service layer
- **Configuration management**: Centralized config with secure token storage
- **Error handling**: Comprehensive exception handling with user-friendly messages
- **Extensibility**: Plugin-style architecture for adding new platforms

## Development Workflow

When working on this codebase:

1. **Configuration Changes**: Modify `git_mcp/core/config.py` for new config options
2. **Platform Support**: Add new adapters in `git_mcp/platforms/` following the base interface
3. **MCP Tools**: Add new tools in `git_mcp/mcp_server.py` with proper async patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yumeminami/git_mcp](https://github.com/yumeminami/git_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
