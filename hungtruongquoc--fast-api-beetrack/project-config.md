---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Running the Application
- `./scripts/run.sh dev` - Start development server with auto-reload
- `./scripts/run.sh prod` - Start production server with multiple workers
- `make dev` - Alternative way to start development server

### Testing
- `./scripts/run.sh test` - Run all tests with verbose output
- `./scripts/run.sh test-fast` - Run tests with minimal output
- `./scripts/run.sh test-cov` - Run tests with coverage report
- `./scripts/run.sh test-cov-html` - Generate HTML coverage report
- `poetry run pytest tests/test_main.py -v` - Run specific test file
- `poetry run pytest tests/api/v1/endpoints/test_items.py::TestItemsAPI -v` - Run specific test class

### Code Quality
- `./scripts/run.sh format` - Format code with Black
- `./scripts/run.sh lint` - Run Flake8 linter
- `./scripts/run.sh typecheck` - Run MyPy type checker
- `./scripts/run.sh quality` - Run all quality checks (format + lint + typecheck + test)

### Dependencies
- `poetry install` - Install all dependencies
- `poetry add package-name` - Add new dependency
- `poetry add --group dev package-name` - Add development dependency

## Architecture Overview

This FastAPI application follows a **layered architecture** pattern:

### Core Layers
1. **API Layer** (`app/api/v1/`) - HTTP request/response handling, routing, OpenAPI documentation
2. **Service Layer** (`app/services/`) - Business logic, data operations, reusable across endpoints
3. **Schema Layer** (`app/schemas/`) - Pydantic models for validation, serialization, type checking
4. **Core Layer** (`app/core/`) - Configuration management, settings, logging utilities
5. **Middleware Layer** (`app/middleware/`) - Request/response processing, logging context injection

### Key Patterns
- **Dependency Injection**: Services are injected via FastAPI's Depends() system
- **Pydantic Validation**: All request/response data is validated using Pydantic v2 models
- **Singleton Services**: Services use singleton pattern for shared state management
- **API Versioning**: All endpoints are versioned under `/api/v1/` prefix
- **OAuth Authentication**: Token-based authentication with automatic token refresh
- **HTTP Client Abstraction**: Centralized HTTP client service for external API calls
- **Token Caching**: In-memory token caching with expiration buffer for performance

### File Structure
```
app/
├── main.py                    # FastAPI app instance, CORS, root endpoints
├── api/v1/                   # API version 1
│   ├── router.py             # Main router aggregating all endpoints
│   └── endpoints/            # Individual resource endpoints
├── services/                 # Business logic layer
│   ├── item_service.py       # Item business operations
│   ├── http_client_service.py # HTTP client for external API calls
│   ├── oauth_authentication_service.py # OAuth token management
│   └── auth_token_cache_service.py # Token caching and refresh
├── schemas/                  # Pydantic models
├── core/                     # Configuration and utilities
│   ├── config.py             # Application settings
│   └── logging.py            # Structured logging configuration
└── middleware/               # Request/response processing
    └── logging.py            # Request context injection
```

## Configuration

- **Settings**: Managed via `app/core/config.py` using Pydantic Settings
- **Environment Variables**: Loaded from `.env` file or system environment
- **CORS**: Configured in `app/main.py` with allowed origins from settings
- **Poetry**: All dependencies managed in `pyproject.toml`

## Testing Approach

- **Test Structure**: Mirrors app structure (`tests/api/`, `tests/services/`, etc.)
- **Test Framework**: pytest with asyncio support
- **Test Client**: Uses HTTPX TestClient for API integration tests
- **Fixtures**: Shared test data and setup in `tests/conftest.py`
- **Coverage**: 77+ tests covering all layers

## Development Workflow

1. **Virtual Environment**: Always ensure `venv` is activated
2. **Code Style**: Use Black formatter (line length 88)
3. **Type Checking**: MyPy configured for Python 3.9+
4. **Linting**: Flake8 for code quality
5. **Testing**: Write tests for new features in appropriate test modules

## Adding New Features

### New API Endpoint
1. Create endpoint function in `app/api/v1/endpoints/`
2. Create Pydantic schemas in `app/schemas/`
3. Add business logic to service in `app/services/`
4. Register endpoint in `app/api/v1/router.py`
5. Write tests in corresponding `tests/` directory

### New Service
1. Create service class in `app/services/`
2. Implement dependency function for injection
3. Add service methods with type hints
4. Write unit tests in `tests/services/`

## Logging

This application uses **structured logging** with `structlog` for better observability and debugging.

### Configuration
- **Development**: Human-readable console output with colors
- **Production**: JSON formatted logs for log aggregation tools
- **Environment Variables**: 
  - `ENVIRONMENT=development|production` - Controls log format
  - `LOG_LEVEL=DEBUG|INFO|WARNING|ERROR` - Sets minimum log level


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hungtruongquoc/fast-api-beetrack](https://github.com/hungtruongquoc/fast-api-beetrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
