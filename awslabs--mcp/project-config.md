---
trigger: always_on
description: This is the **AWS DynamoDB MCP Server** - an official AWS Labs Model Context Protocol (MCP) server that provides DynamoDB expert design guidance and data modeling assistance. The project is built with Python 3.10+ and uses `uv` for dependency management.
---

# AGENTS.md

## Project Overview

This is the **AWS DynamoDB MCP Server** - an official AWS Labs Model Context Protocol (MCP) server that provides DynamoDB expert design guidance and data modeling assistance. The project is built with Python 3.10+ and uses `uv` for dependency management.

**Current Version**: See `version` in [pyproject.toml](pyproject.toml)

**Project URLs**:
- Homepage: https://awslabs.github.io/mcp/
- Documentation: https://awslabs.github.io/mcp/servers/dynamodb-mcp-server/
- Repository: https://github.com/awslabs/mcp.git
- Changelog: https://github.com/awslabs/mcp/blob/main/src/dynamodb-mcp-server/CHANGELOG.md

**Package Information**:
- PyPI Package: `awslabs.dynamodb-mcp-server`
- License: Apache-2.0

## Setup Commands

### Prerequisites
- Install `uv` from [Astral](https://docs.astral.sh/uv/getting-started/installation/)
- Install Python: `uv python install 3.10`
- Set up AWS credentials with access to AWS services

### Development Environment
```bash
# Install dependencies
uv sync

# Install development dependencies
uv sync --group dev

# Activate virtual environment
source .venv/bin/activate

# Run the MCP server
uv run awslabs.dynamodb-mcp-server

# Run with uvx (production-like)
uvx awslabs.dynamodb-mcp-server@latest
```

### Docker Development
```bash
# Build Docker image
docker build -t awslabs/dynamodb-mcp-server .

# Run Docker container
docker run --rm --interactive --env FASTMCP_LOG_LEVEL=ERROR awslabs/dynamodb-mcp-server:latest

# Docker healthcheck
# The container includes a healthcheck script at /app/docker-healthcheck.sh
```

## Code Style and Quality

### Quality Tools
```bash
# Format code
uv run ruff format

# Lint code
uv run ruff check

# Fix linting issues automatically
uv run ruff check --fix

# Type checking
uv run pyright

# Run all quality checks
uv run ruff check && uv run pyright
```

### Code Style Configuration
- **Formatter**: Ruff (see pyproject.toml for complete configuration)
- **Type Checker**: Pyright (configured in pyproject.toml)
- Complete style rules and exceptions are defined in pyproject.toml

### Pre-commit Setup
```bash
# Install pre-commit hooks (if .pre-commit-config.yaml exists)
uv run pre-commit install

# Run pre-commit on all files
uv run pre-commit run --all-files
```

**Note**: This project includes pre-commit as a dev dependency but does not have a `.pre-commit-config.yaml` file configured.

## Testing

### Test Execution
```bash
# Run all tests
uv run pytest

# Run tests with coverage
uv run pytest --cov=awslabs --cov-report=html

# Run specific test file
uv run pytest tests/test_dynamodb_server.py

# Run with verbose output
uv run pytest -v

# Run specific test function
uv run pytest tests/test_dynamodb_server.py::test_function_name

# Run tests by marker
uv run pytest -m integration  # Run integration tests
uv run pytest -m "not live"   # Skip live tests (default behavior)
uv run pytest -m unit         # Run unit tests only
```

### Test Categories and Markers
The project uses pytest markers to categorize tests (configured in pyproject.toml):
- **integration**: Integration tests (slower, end-to-end)
- **live**: Live API calls (skipped by default)
- **asyncio**: Async tests (auto-mode enabled)
- **unit**: Unit tests (fast, isolated)
- **file_generation**: File generation tests
- **slow**: Comprehensive/slow tests
- **python**: Python language-specific tests
- **snapshot**: Snapshot tests for generated code consistency

**Default Test Behavior**: Tests marked with `integration` or `live` are excluded by default (configured via pytest addopts: `-m 'not integration and not live'`)

### Test Suite
- **Property-based tests**: Using `hypothesis` for comprehensive input validation
- **Comprehensive test coverage**: Unit and integration tests
- **Async test support**: pytest-asyncio with auto mode
- **Mocking support**: Using `moto` for AWS service mocking
- **Coverage exclusions**: Pragma comments and main blocks are excluded

### Available Test Files and Directories
- `tests/test_dynamodb_server.py` - Main MCP server tests
- `tests/test_common.py` - Common utilities tests
- `tests/test_markdown_formatter.py` - Markdown formatting tests
- `tests/test_model_validation_utils.py` - DynamoDB validation tests
- `tests/db_analyzer/` - Database analyzer tests
- `tests/cdk_generator/` - CDK code generation tests
- `tests/repo_generation_tool/` - Data access layer generation tests
- `tests/conftest.py` - Shared pytest fixtures and configuration

### Test Environment Setup
- Tests use `pytest` with `asyncio_mode = "auto"` (configured in pyproject.toml)
- MySQL integration tests use environment variable fixtures (mysql_env_setup)
- Coverage reports exclude pragma comments and main blocks (configured in pyproject.toml)
- Coverage source: `awslabs` directory
- Coverage omits: `awslabs/dynamodb_mcp_server/repo_generation_tool/languages/python/base_repository.py`

## Project Structure

### Core Components
- `awslabs/dynamodb_mcp_server/server.py` - Main MCP server implementation with FastMCP

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awslabs/mcp](https://github.com/awslabs/mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
