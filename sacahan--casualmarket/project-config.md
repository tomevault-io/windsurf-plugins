---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CasualMarket is a Taiwan Stock Exchange MCP (Model Context Protocol) Server that provides real-time stock price queries for Taiwan securities. It uses FastMCP framework for simplified MCP tool registration and includes intelligent rate limiting, caching, and financial analysis features.

## Development Commands

### Server Execution

CasualMarket supports two deployment modes:
1. **stdio mode**: Local development and Claude Desktop integration
2. **Docker + SSE mode**: Containerized deployment with HTTP interface

#### Production Execution

```bash
# stdio mode (local/desktop)
uvx --from . casual-market-mcp

# Docker mode (SSE HTTP interface)
docker-compose up -d
```

#### Development Execution (Recommended for Active Development)

```bash
# Development mode - direct execution avoids uvx cache issues
uv run python -m src.main

# Quick functionality test
uv run python tests/api/demo_enhanced_client.py
```

#### Cache Management

```bash
# Clear uvx cache when needed (production deployment)
uv cache clean

# Development mode automatically uses latest code
# No cache clearing needed when using uv run
```

### Docker Deployment

```bash
# Pull and start container (recommended)
./scripts/docker-run.sh pull
./scripts/docker-run.sh up

# Or build locally
./scripts/docker-run.sh build
DOCKER_IMAGE_NAME=casualmarket-mcp:latest ./scripts/docker-run.sh up

# View logs
./scripts/docker-run.sh logs

# Test service
./scripts/docker-run.sh test

# Stop and remove container
./scripts/docker-run.sh down

# Multi-platform build and push to Docker Hub
./scripts/build_docker.sh --platform all --action build-push


```

**Docker Endpoints:**
- Root: `http://localhost:8000/`
- Health: `http://localhost:8000/health`
- SSE: `http://localhost:8000/sse` (MCP protocol)
- Docs: `http://localhost:8000/docs`

**Quick Commands:**
```bash
# View service info
./scripts/docker-run.sh info

# Enter container shell
./scripts/docker-run.sh shell

# Restart service
./scripts/docker-run.sh restart

# Complete cleanup
./scripts/docker-run.sh clean
```

**Example SSE Client:**
```bash
# Run example client
python examples/sse_client_example.py
```

### Testing

```bash
# Run all tests with coverage
uv run pytest

# Run specific test categories
uv run pytest tests/api/           # API integration tests
uv run pytest tests/server/        # Server functionality tests
uv run pytest tests/mcp/           # MCP protocol tests
uv run pytest tests/tools/         # Tool functionality tests

# Run a single test file
uv run pytest tests/api/test_twse_standalone.py

# Generate coverage report
uv run pytest --cov=src --cov-report=html
```

#### Current Testing Status

**Overall Health**: ✅ Excellent (98% pass rate)

- **Total Test Cases**: 110
- **Passing Tests**: 108 (98%)
- **Skipped Tests**: 2 (2%)
- **Failed Tests**: 0 (0%)
- **Code Coverage**: 62%

**Test Categories**:

- ✅ **API Integration Tests** (`tests/api/`) - All passing, includes rate limiting and caching
- ✅ **Server Functionality** (`tests/server/`) - All core server features working
- ✅ **MCP Protocol Tests** (`tests/mcp/`) - Protocol compliance verified
- ✅ **Tool Functionality** (`tests/tools/`) - All tools fully tested and working
  - ✅ Foreign Investment Tools (12/12 passing)
  - ✅ Market Tools (13/13 passing)
  - ✅ Financial Tools (all passing)
  - ✅ Trading Tools (all passing)

**Recent Fixes**: Successfully resolved 19 failing test cases in foreign investment and market tools by:

- Correcting mock API method calls to match actual implementations
- Aligning test expectations with actual tool return formats
- Fixing parameter names and data structures
- Ensuring proper error message validation

### Code Quality

```bash
# Lint with ruff
uv run ruff check src/ tests/

# Type checking with mypy
uv run mypy src/

# Fix auto-fixable linting issues
uv run ruff check --fix src/ tests/
```

### Development Setup Verification

```bash
# Test uvx execution and MCP protocol
./tests/test_uvx_execution.sh

# Test specific enhanced client functionality
uv run python tests/api/demo_enhanced_client.py

# Debug API functionality
uv run python tests/api/debug_api.py
```

## Architecture Overview

### Core Components

1. **FastMCP Server** (`src/server.py`): Main MCP server using `@mcp.tool` decorators for simplified tool registration
2. **API Client Layer** (`src/api/`):
   - `twse_client.py`: Taiwan Stock Exchange API integration with decorator-based enhancements
   - `openapi_client.py`: TWSE OpenAPI integration for financial statements
   - `decorators.py`: Function decorators for adding caching and rate limiting to API methods
3. **Cache System** (`src/cache/`): Integrated rate limiting and caching service with request tracking
4. **Securities Database** (`src/securities_db.py`): SQLite database for ISIN codes and company name resolution
5. **Tool Base Architecture** (`src/tools/base/`): Base classes and decorators for standardized tool development
6. **Financial Tools** (`src/tools/`): Domain-organized tools including:
   - `trading/`: Stock prices, trading operations, statistics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sacahan/CasualMarket](https://github.com/sacahan/CasualMarket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
