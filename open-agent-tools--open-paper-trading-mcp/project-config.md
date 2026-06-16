---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Open Paper Trading MCP is a comprehensive paper trading simulator with dual interfaces: a REST API (FastAPI) and AI agent tools (MCP). The system simulates multi-asset trading (stocks, options, ETFs, bonds) with real market data for algorithmic trading development and AI agent training.

**CRITICAL: Real Data Only Policy** - The system MUST always use real market data from Robinhood API for all production APIs, MCP tools, and core trading service responses. Synthetic/test data should ONLY be used in pytest mocks and test fixtures. Never use test adapters in production Docker containers or live services.

**Current Status (2025-08-06)**: 🎉 **PRODUCTION READY QUALITY** - Successfully implemented dual-server architecture with FastAPI server (port 2080) for frontend/API and independent MCP server (port 2081) for AI agent tools. Both servers running simultaneously with full functionality. FastMCP integration resolved via server separation after mounting conflicts. **Code cleanup completed**: All ruff linting issues resolved (100% compliance), core application 100% mypy compliant, 576/581 journey tests passing (99.1% success rate). AsyncIO infrastructure fully stabilized with zero warnings. Database connection pool optimized with proper resource management. **Advanced Options Trading**: Professional spread builder with 15+ strategies, real-time P&L analysis, and comprehensive risk metrics. **MCP Tool Validation Complete**: 42/42 ADK evaluations tested with 100% agent behavior validation - all agents correctly execute proper multi-step workflows using live market data.

## Essential Commands

### Development Commands
```bash
# Run all development tasks via the dev script
python scripts/dev.py <command>

# Available commands:
python scripts/dev.py server     # Start FastAPI server only (port 2080)
python scripts/dev.py test       # Run all tests (AVOID - use journey-based testing instead)
python scripts/dev.py format     # Format code (uv run ruff format .)
python scripts/dev.py lint       # Lint code (uv run ruff check . --fix)
python scripts/dev.py typecheck  # Type check (uv run mypy .)
python scripts/dev.py check      # Run all checks (format + lint + typecheck + tests)
```

### Direct Commands
```bash
# Servers (split architecture)
uv run python app/main.py        # Start FastAPI server (port 2080)
uv run python app/mcp_server.py  # Start MCP server (port 2081)

# Testing - PREFER USER JOURNEY-BASED TESTING
uv run pytest -v                 # All tests (AVOID - causes timeouts with 581 tests)
pytest tests/unit/               # Unit tests only
pytest tests/integration/        # Integration tests
pytest tests/performance/        # Performance tests (if directory exists)
pytest -m "not slow"             # Skip slow tests
pytest -m "database"             # Database tests only

# User Journey Testing (see User Journey-Based Test Organization section below)
pytest -m "journey_account_management"      # Account setup & management (69 tests)
pytest -m "journey_account_infrastructure"  # Account adapters, filesystem, error handling (114 tests)
pytest -m "journey_basic_trading"           # Stock orders & portfolio (73 tests)
pytest -m "journey_market_data"             # Quotes & market data (76 tests)
pytest -m "journey_options_trading"         # Basic options & Greeks (79 tests)
pytest -m "journey_options_advanced"        # Advanced options, multi-leg strategies (72 tests)
pytest -m "journey_complex_strategies"      # Complex strategies (72 tests)
pytest -m "journey_performance"             # Performance & optimization (78 tests)
pytest -m "journey_integration"             # End-to-end & live API (59 tests)

# Code Quality
uv run ruff check . --fix        # Lint and auto-fix issues
uv run ruff format .             # Format code (replaces black + isort)
uv run mypy .                    # Type checking

# Database Setup (required for tests)
python3 scripts/setup_test_db.py        # Set up separate test database (trading_db_test)
python3 scripts/setup_test_db.py cleanup # Clean up test database
```

### Docker Commands
```bash
docker-compose up --build        # Start all services
docker-compose up -d             # Start in background (required for tests)
```

## Architecture Overview

### Core Architecture Pattern
- **Split Server Architecture**: FastAPI server (port 2080) and independent MCP server (port 2081)
- **Simplified Direct Connection**: TradingService connects directly to PostgreSQL and Robinhood API
- **No Message Queue/Cache**: Direct database operations for all trading state
- **Dual Interface**: Separate servers for REST API and MCP tools
- **Async Throughout**: All operations use asyncio for performance

### Key Components

**App Structure:**
```
app/
├── main.py                 # FastAPI server startup (port 2080)
├── mcp_server.py           # Independent MCP server (port 2081)
├── mcp_tools.py            # MCP tools using FastMCP framework
├── core/                   # Configuration, logging, exceptions, dependencies
├── api/                    # REST API routes (FastAPI)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Open-Agent-Tools/open-paper-trading-mcp](https://github.com/Open-Agent-Tools/open-paper-trading-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
