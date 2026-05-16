---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Official API Documentation

**Anytype API Documentation**: https://developers.anytype.io/docs/reference/
- **Current API Version**: 2025-05-20
- **Properties API**: https://developers.anytype.io/docs/reference/2025-05-20/list-properties
- **Key Note**: Properties are EXPERIMENTAL and may change in future updates ⚠️

**Property Endpoints**:
- `GET /v1/spaces/:space_id/properties` - List properties
- `POST /v1/spaces/:space_id/properties` - Create property  
- `GET /v1/spaces/:space_id/properties/:property_id` - Get property
- `PATCH /v1/spaces/:space_id/properties/:property_id` - Update property (requires `name` field)
- `DELETE /v1/spaces/:space_id/properties/:property_id` - Delete property (actually archives it)

## Architecture Overview

This is a Python client library for the Anytype API. The codebase follows a layered architecture:

- **Client Layer** (`anytype_client/client.py`): Contains both synchronous (`AnytypeClient`) and asynchronous (`AsyncAnytypeClient`) HTTP clients that handle API authentication, request processing, and response parsing
- **Model Layer** (`anytype_client/models.py`): Pydantic models for API request/response serialization, including core entities like `Space`, `Object`, `ObjectTypeDefinition`, and various supporting models
- **Exception Layer** (`anytype_client/exceptions.py`): Custom exception hierarchy for different types of API errors (authentication, validation, rate limiting, etc.)

## Key Components

### Authentication
- API key-based authentication using Bearer tokens
- Interactive authentication flow with challenge codes
- API key is read from `ANYTYPE_API_KEY` environment variable or passed directly
- Default base URL is `http://localhost:31009/v1/` (local Anytype instance)

### Clients
- **BaseClient**: Generic base class with shared functionality
- **AnytypeClient**: Synchronous client with context manager support
- **AsyncAnytypeClient**: Asynchronous client with async context manager support

### Core Models
- **Space**: Represents Anytype workspaces with properties like `name`, `description`, `gateway_url`, `network_id`
- **Object**: Represents Anytype objects with properties, relations, and metadata
- **ObjectTypeDefinition**: Custom object type definitions
- **Property**: Property definitions with formats and constraints
- **List**: Represents lists in Anytype with items and metadata
- **Member**: Represents workspace members with roles and permissions
- **Tag**: Represents tags with colors and descriptions
- **Template**: Represents object templates with type and content definitions

### API Response Handling
- Robust error handling with specific exceptions for different HTTP status codes
- Response parsing with Pydantic validation
- Special handling for paginated responses (spaces API returns `{data: [...], pagination: {...}}`)

## Common Development Commands

### Installation

This project uses Poetry for dependency management:

```bash
# Install Poetry if needed
curl -sSL https://install.python-poetry.org | python3 -

# Install dependencies
poetry install

# Activate environment
poetry shell
```

Alternative with pip:
```bash
pip install -e .
pip install -e .[dev]  # For development dependencies
```

### Development Dependencies
- `pytest>=7.0.0` - Testing framework
- `black>=23.0.0` - Code formatting
- `isort>=5.12.0` - Import sorting
- `mypy>=1.0.0` - Type checking
- `pytest-asyncio>=0.21.0` - Async testing support
- `ruff>=0.1.0` - Fast Python linter
- `pre-commit>=3.0.0` - Git hooks for code quality

### Code Quality Commands

With Poetry (recommended):
```bash
poetry run black anytype_client/  # Format code
poetry run isort anytype_client/  # Sort imports
poetry run mypy anytype_client/   # Type checking
poetry run ruff check anytype_client/  # Linting
poetry run pytest                # Run tests
```

With activated environment:
```bash
poetry shell  # or source anytype-python-client-venv/bin/activate
black anytype_client/
isort anytype_client/
mypy anytype_client/
ruff check anytype_client/
pytest
```

### Testing Commands (IMPORTANT - Remember this!)

With Poetry:
```bash
poetry run python -m pytest tests/test_objects.py::TestObjectCRUD::test_create_object -v  # Run specific test
poetry run python run_tests.py --quick    # Run quick test suite
poetry run python run_tests.py --all      # Run all tests
poetry run pytest tests/ -v              # Run all tests with pytest directly
```

With virtual environment (legacy):
```bash
# Activate the virtual environment first
source anytype-python-client-venv/bin/activate

# Then run tests
python -m pytest tests/test_objects.py::TestObjectCRUD::test_create_object -v  # Run specific test
python run_tests.py --quick    # Run quick test suite
python run_tests.py --all      # Run all tests
python -m pytest tests/ -v    # Run all tests with pytest directly
```

### API Endpoints Format (IMPORTANT)
The Anytype API uses **space-scoped endpoints** for objects:
- ✅ Correct: `/v1/spaces/{space_id}/objects` 
- ❌ Wrong: `/v1/objects`

Object creation requires:
- `type_key` field (not `type`)
- API responses have nested format: `{"object": {...}}`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [beaucronin/anytype-python-client](https://github.com/beaucronin/anytype-python-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
