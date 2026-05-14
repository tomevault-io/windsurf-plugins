---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

REE MCP Server: A production-ready MCP (Model Context Protocol) server for accessing Red Eléctrica Española (REE) electricity data through the eSios API. Built with strict Domain-Driven Design (DDD) principles, Clean Architecture, and comprehensive testing.

**Key Constraint**: This codebase follows a NO MOCKING policy in the domain layer. Domain logic must remain pure and testable without mocks.

## Development Commands

### Environment Setup
```bash
# Using uv (recommended)
uv venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
uv pip install -e ".[dev]"

# Using standard pip
python -m venv venv
source venv/bin/activate
pip install -e ".[dev]"
```

### Testing
```bash
# Run all tests
pytest

# Run only unit tests (domain layer)
pytest tests/unit/

# Run integration tests (infrastructure with mocked HTTP)
pytest tests/integration/

# Run e2e tests
pytest tests/e2e/

# Run with coverage report
pytest --cov=src/ree_mcp --cov-report=html

# Run specific test file
pytest tests/unit/domain/test_value_objects.py

# Run specific test function
pytest tests/unit/domain/test_value_objects.py::test_indicator_id_validation
```

### Type Checking
```bash
# Type check entire codebase (mypy strict mode enabled)
mypy src/ree_mcp/

# Type check specific module
mypy src/ree_mcp/domain/
```

### Linting and Formatting
```bash
# Check code style
ruff check .

# Auto-fix issues
ruff check --fix .

# Format code (line length: 100)
ruff format .
```

### Running the Server
```bash
# STDIO mode (default for MCP)
python -m ree_mcp

# HTTP mode for testing/debugging
python -c "from ree_mcp.interface.mcp_server import mcp; mcp.run(transport='http', port=8000)"
```

## Architecture Overview

This project implements **Clean Architecture** with **Domain-Driven Design (DDD)** in four distinct layers:

### 1. Domain Layer (`src/ree_mcp/domain/`)
**Pure business logic with ZERO external dependencies. No I/O, no frameworks, no mocking in tests.**

- **Value Objects** (`value_objects/`): Immutable domain concepts
  - `IndicatorId`: Strongly-typed IDs with validation (must be > 0)
  - `DateTimeRange`: Date ranges with business rules (max 366 days, start < end)
  - `TimeGranularity`: Enum for aggregation levels (raw/hour/day/fifteen_minutes)
  - `MeasurementUnit`: Power (MW), Price (€/MWh), Emissions (tCO₂eq)
  - `GeographicScope`: Peninsular, National, Canarias, etc.

- **Entities** (`entities/`): Business objects with identity
  - `Indicator`: Electricity indicator with metadata + helper methods (is_demand, is_generation)
  - `IndicatorValue`: Single time-series data point
  - `IndicatorData`: **Aggregate Root** - indicator + values + computed statistics

- **Repository Interface** (`repositories/`): Abstract contract defined in domain
  - `IndicatorRepository`: Interface for data access (implemented in infrastructure)

- **Domain Exceptions** (`exceptions.py`): Business rule violations
  - `InvalidIndicatorIdError`, `InvalidDateRangeError`
  - `IndicatorNotFoundError`, `NoDataAvailableError`

**Critical Rule**: Domain tests (`tests/unit/domain/`) test pure logic without mocks, HTTP, or external dependencies.

### 2. Application Layer (`src/ree_mcp/application/`)
**Orchestrates domain objects to fulfill use cases. Depends only on domain.**

- **Use Cases** (`use_cases/`): Business workflows
  - `GetIndicatorDataUseCase`: Fetch time-series data via repository
  - `ListIndicatorsUseCase`: Retrieve all indicators
  - `SearchIndicatorsUseCase`: Find indicators by keyword

- **DTOs** (`dtos/`): Data Transfer Objects for boundaries
  - `GetIndicatorDataRequest`: Input validation with Pydantic
  - `IndicatorDataResponse`: Structured output
  - Uses Pydantic for validation and serialization

### 3. Infrastructure Layer (`src/ree_mcp/infrastructure/`)
**Implements domain interfaces. Handles external dependencies.**

- **HTTP Client** (`http/ree_api_client.py`):
  - Async httpx client with context manager
  - **Exponential backoff retry** for transient failures (HTTP 500, timeouts)
  - Configurable via Settings (timeout, max_retries, backoff_factor)

- **Repository Implementation** (`repositories/ree_indicator_repository.py`):
  - Implements `IndicatorRepository` interface
  - Maps API responses to domain entities
  - Handles data parsing and transformation

- **Configuration** (`config/settings.py`):
  - Pydantic-settings for type-safe env config
  - Loads from `.env` file with defaults
  - Required: `REE_API_TOKEN`
  - Optional: `REE_API_BASE_URL`, `REQUEST_TIMEOUT`, `MAX_RETRIES`, `RETRY_BACKOFF_FACTOR`

**Integration Tests** (`tests/integration/`): Test infrastructure with mocked HTTP (pytest-httpx).

### 4. Interface Layer (`src/ree_mcp/interface/`)
**Exposes domain functionality via MCP protocol. Refactored for maintainability following DRY, KISS, and SOLID principles.**

- **MCP Server** (`mcp_server.py`): FastMCP integration (923 lines, 30% reduction from original)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ESJavadex/ree-mcp](https://github.com/ESJavadex/ree-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
