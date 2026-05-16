---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Article MCP is a high-performance literature search server based on FastMCP framework that integrates multiple academic databases including Europe PMC, arXiv, PubMed, CrossRef, and OpenAlex. It provides comprehensive literature search, reference management, and quality evaluation tools for academic research.

## Architecture

The project follows a standard Python src layout with layered architecture:

- **CLI Layer** (`cli.py`): Main CLI entry point and MCP server creation via `create_mcp_server()`
- **Tool Layer** (`tools/core/`): 6 core MCP tool registrations
- **Service Layer** (`services/`): API integrations using dependency injection pattern
- **Middleware Layer** (`middleware/`): Error handling, logging, and performance monitoring
- **Resource Layer** (`resources/`): MCP resources for config and journal data
- **Compatibility Layer** (`main.py`): Backward-compatible CLI entry point

### The 5 Core Tools

All tools are registered in `cli.py:create_mcp_server()` with their respective service dependencies:

1. **search_literature** (`tools/core/search_tools.py`): Multi-source literature search
2. **get_article_details** (`tools/core/article_tools.py`): Article details by identifier
3. **get_references** (`tools/core/reference_tools.py`): Reference list retrieval
4. **get_literature_relations** (`tools/core/relation_tools.py`): Citation relationship analysis
5. **get_journal_quality** (`tools/core/quality_tools.py`): Journal quality assessment

Note: The test file `test_six_tools.py` is a legacy name - it tests these 5 core tools.

### Service Injection Pattern

Services are created in `create_mcp_server()` and passed to tool registrars:
```python
pubmed_service = create_pubmed_service(logger)
europe_pmc_service = create_europe_pmc_service(logger, pubmed_service)
# ... other services
register_search_tools(mcp, {"europe_pmc": europe_pmc_service, ...}, logger)
```

### Middleware System

Three middlewares are added to the MCP server in `create_mcp_server()`:

- **MCPErrorHandlingMiddleware** (`middleware/`): Converts exceptions to MCP standard errors
- **LoggingMiddleware** (`middleware/logging.py`): Request/response logging
- **TimingMiddleware** (`middleware/logging.py`): Performance stats collection

Access global performance stats via `middleware.get_global_performance_stats()`.

## Development Commands

### Environment Setup
```bash
# Install dependencies
uv sync

# Or using pip
pip install fastmcp requests python-dateutil aiohttp markdownify
```

### Running the Server
```bash
# Production (recommended)
uvx article-mcp server

# Local development - using new CLI
uv run python -m article_mcp server

# Compatibility through main.py (still works)
uv run main.py server

# Alternative transport modes
uv run python -m article_mcp server --transport stdio
uv run python -m article_mcp server --transport sse --host 0.0.0.0 --port 9000
uv run python -m article_mcp server --transport streamable-http --host 0.0.0.0 --port 9000
```

### Testing
```bash
# Core functionality (recommended for daily use)
uv run python scripts/test_working_functions.py

# Quick validation
uv run python scripts/quick_test.py

# Complete test suite
uv run python scripts/run_all_tests.py

# Individual tests
uv run python scripts/test_basic_functionality.py
uv run python scripts/test_cli_functions.py
uv run python scripts/test_service_modules.py
uv run python scripts/test_integration.py
uv run python scripts/test_performance.py

# Pytest-based tests
pytest                    # Run all tests
pytest tests/unit/        # Unit tests only
pytest -m integration     # Integration tests only
pytest -m "not slow"      # Exclude slow tests
pytest tests/unit/test_six_tools.py  # Test all 5 core tools (legacy filename)
```

### Code Quality
```bash
# Format code (Ruff formatter - 100x faster than black)
ruff format src/ tests/

# Lint (Ruff linter - replaces flake8, isort)
ruff check src/ tests/
ruff check src/ tests/ --fix  # Auto-fix issues

# Type checking
mypy src/

# All quality checks
ruff check src/ tests/ --fix && mypy src/ && ruff format --check src/ tests/

# Pre-commit hooks (install)
pre-commit install
```

### Package Management
```bash
python -m build           # Build package
uvx --from . article-mcp server  # Install from local
uvx article-mcp server      # Test PyPI package
```

## Key Development Patterns

### TDD Approach

New features should be implemented using TDD methodology:
1. Write test file first in `tests/unit/`
2. Implement feature to pass tests
3. Example: `OpenAlexMetricsService` was implemented with `test_openalex_metrics_service.py` (15 tests)

### Caching Strategy

The project implements intelligent caching with 24-hour expiry:
- Cache file: `.cache/journal_quality/journal_data.json` (unified cache)
- OpenAlex and EasyScholar share the same cache file with merged data structure:
  ```json
  {
    "journals": {
      "Nature": {
        "data": {...},           // EasyScholar data
        "openalex_metrics": {...}, // OpenAlex metrics
        "timestamp": 1234567890.0
      }
    }
  }
  ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gqy20/article-mcp](https://github.com/gqy20/article-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
