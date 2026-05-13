---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Multi-MCP is a multi-model AI orchestration server that provides advanced code analysis capabilities through the Model Context Protocol (MCP). It orchestrates multiple LLM providers via LiteLLM to deliver systematic code review and analysis tools.

The server is built with FastMCP and uses a streamlined workflow architecture optimized for fast, cost-effective analysis with models like gpt-5-mini.

## Current Status

**Production Ready** ✅
- **Unit Tests**: ✅ 511 tests passing (~2s) - All tests passing (includes 24 mocked CLI tests for parsing/error handling)
- **Integration Tests**: ✅ 93 tests passing (~8-10min) - All tests passing (includes 6 CLI smoke tests, 8 CLI workflow tests)
- **Total Coverage**: ✅ 604 tests passing (~85% code coverage)
- **Model Config**: YAML-based model configuration with aliases and use-case defaults
- **Logging**: MCP tool request/response logging enabled
- **Implementation**: Checklist-based workflow with expert validation enabled
- **File Limit Enforcement**: ✅ `settings.max_files_per_review` is enforced

## Development Commands

```bash
# Type checking (required before commits)
uv run pyright

# Linting and formatting (required before commits)
uv run ruff check .
uv run ruff format .

# Run all unit tests (511 tests, ~2s, all passing ✅)
uv run pytest tests/unit/ -v

# Run integration tests (93 tests, ~5-7min with parallel, all passing ✅)
# Note: Requires real API keys (OPENAI_API_KEY, etc.)
# CLI tests will skip gracefully if CLIs not installed
RUN_E2E=1 uv run pytest tests/integration/ -n auto -v

# Or run sequentially (slower, ~15min)
RUN_E2E=1 uv run pytest tests/integration/ -v

# Run all tests (604 total)
RUN_E2E=1 uv run pytest tests/ -v

# Run the MCP server
./scripts/run_server.sh
# or: uv run python multi_mcp/server.py

# View MCP logs (request/response)
ls -lh logs/*.mcp.json
cat logs/*.mcp.json | jq .
```

## Building & Publishing to PyPI

**Package**: https://pypi.org/project/multi-mcp/

```bash
# Build package (creates dist/*.whl and dist/*.tar.gz)
make build

# Publish to TestPyPI (for testing)
make publish-test

# Publish to PyPI (production)
make publish

# Clean build artifacts before rebuilding
make clean
```

**Version Management:**
- Version is in `pyproject.toml` (line 3): `version = "X.Y.Z"`
- Bump version before publishing (PyPI doesn't allow re-uploading same version)
- Entry points: `multi` (CLI), `multi-server` (MCP server), `multi-mcp` (MCP server alias for uvx)

**User Installation:**
```bash
pip install multi-mcp
claude mcp add multi -- uvx multi-mcp
```

**Automated Publishing:** See `docs/github-pypi-v1.md` for GitHub Actions workflow plan.

## Installation & Setup

See README.md for installation instructions and environment setup.

## CLI Usage

See README.md for CLI usage examples. Note: CLI is experimental.

## Architecture

### Core Components

**`multi_mcp/server.py`**: FastMCP server implementation with factory-generated tool wrappers
- Uses `create_mcp_wrapper()` factory to auto-generate tools from schemas
- Tool wrappers decorated with `@mcp.tool()` and `@mcp_monitor` for logging
- Calls `*_impl()` functions from `multi_mcp/tools/` for actual implementation

**`multi_mcp/tools/`**: Tool implementation functions
- `codereview.py` - Code review workflow with checklist guidance and expert validation
- `chat.py` - Interactive chat for development questions
- `compare.py` - Multi-model parallel analysis
- `debate.py` - Two-step debate workflow (independent + critique)
- `models.py` - Model listing implementation

**`multi_mcp/models/`**: Model configuration and LLM integration
- `config.py` - YAML-based model config with Pydantic validation (`ModelConfig`, `ModelsConfiguration`, `PROVIDERS`)
- `resolver.py` - Model alias resolution with LiteLLM fallback (`ModelResolver`)
- `litellm_client.py` - API model execution via LiteLLM responses API (~260 lines)
- `cli_executor.py` - CLI model execution via subprocess (~270 lines)

**`multi_mcp/config/config.yaml`**: Model definitions
- Canonical model names with LiteLLM model strings
- Aliases (e.g., `mini` → `gpt-5-mini`, `sonnet` → `claude-sonnet-4.6`)
- Temperature constraints per model
- User overrides: `~/.multi_mcp/config.yaml` (optional)

**`multi_mcp/settings.py`**: Environment-based configuration using Pydantic Settings
- API keys loaded from `.env` files (cascading: project .env > ~/.multi_mcp/.env)
- Runtime defaults (`default_model`, `default_model_list`, `default_temperature`)
- Server settings (`max_retries`, `model_timeout_seconds`, etc.)
- `default_model_list`: Default models for multi-model compare (comma-separated or JSON array in .env)

**`multi_mcp/schemas/`**: Pydantic models for request validation
- `base.py` - Base `BaseToolRequest`, `SingleToolRequest`, `ModelResponseMetadata`
- `codereview.py` - `CodeReviewRequest`, `CodeReviewResponse`
- `chat.py` - `ChatRequest`, `ChatResponse`
- `compare.py` - `CompareRequest`, `CompareResponse`
- `debate.py` - `DebateRequest`, `DebateResponse`
- **Single source of truth**: Field descriptions defined once in Pydantic models

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [religa/multi_mcp](https://github.com/religa/multi_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
