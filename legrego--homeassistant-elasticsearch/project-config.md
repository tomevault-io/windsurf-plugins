---
trigger: always_on
description: This is a Home Assistant custom component that publishes Home Assistant events to Elasticsearch clusters. This document provides guidance for AI coding assistants working with this codebase.
---

# Copilot Instructions for homeassistant-elasticsearch

This is a Home Assistant custom component that publishes Home Assistant events to Elasticsearch clusters. This document provides guidance for AI coding assistants working with this codebase.

## Project Overview

- **Purpose**: Publish Home Assistant events to Elasticsearch using the Bulk API
- **Language**: Python (see `pyproject.toml` for version requirements)
- **Key Dependencies**: 
  - Home Assistant (see `pyproject.toml` for version)
  - Elasticsearch client (see `pyproject.toml` and `custom_components/elasticsearch/manifest.json` for version)
- **Architecture**: Home Assistant custom component with config flow support
- **Quality**: Platinum quality scale (per `manifest.json`)

## Repository Structure

```
custom_components/elasticsearch/  - Main integration code
  __init__.py                     - Component initialization
  config_flow.py                  - Configuration flow UI
  es_gateway.py                   - Elasticsearch API gateway
  es_publish_pipeline.py          - Event publishing pipeline
  datastreams/                    - Datastream management
tests/                            - Test suite
docs/                             - Documentation (MkDocs)
scripts/                          - Development scripts
  lint                            - Run linting/formatting
  test                            - Run test suite
```

## Development Workflow

### Setup

This project uses Poetry for dependency management and includes a devcontainer for consistent development:

```bash
pip install poetry  # See project documentation for recommended version
poetry install
```

### Testing

**ALWAYS run tests before and after making changes:**

```bash
poetry run ./scripts/test
# Or inside poetry shell:
poetry shell
./scripts/test
```

Tests use pytest with Home Assistant custom component test framework. Snapshot testing is used extensively - if snapshots need updates, run `./scripts/update_snapshots`.

### Linting

**ALWAYS run linting before committing:**

```bash
poetry run ./scripts/lint
# Or inside poetry shell:
poetry shell
./scripts/lint
```

Linting uses:
- **ruff** for code formatting and linting (configured in `.ruff.toml`)
- Auto-fixes are applied by default (use `--no-fix` to check only)

## Code Style & Conventions

### Python Style

- **Line length**: 110 characters (configured in `.ruff.toml`)
- **Target**: Python version specified in `pyproject.toml` (note: `.ruff.toml` may use a different `target-version` for linter compatibility)
- **Type hints**: Always use type hints. Use `from __future__ import annotations` for modern syntax
- **Imports**: Managed by ruff/isort - imports are auto-sorted
- **Docstrings**: Required for public APIs (D-series rules enabled)
- **Logging**: Use structured logging via the custom logger module

### Naming Conventions

- **Classes**: PascalCase (e.g., `ElasticIntegration`, `ESGateway`)
- **Functions/Methods**: snake_case (e.g., `async_setup_entry`, `publish_events`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `ELASTIC_DOMAIN`, `CONF_URL`)
- **Private members**: Prefix with underscore (e.g., `_logger`, `_client`)

### Async Patterns

This is an async-first codebase using Home Assistant's async patterns:

- Use `async def` for all I/O operations
- Use `asyncio` primitives, not threading
- Entry point functions follow HA naming: `async_setup_entry`, `async_unload_entry`
- Use `@async_log_enter_exit_info` and `@async_log_enter_exit_debug` decorators from logger module

### Error Handling

Custom exceptions are defined in `errors.py`:
- `CannotConnect` - Connection failures
- `AuthenticationRequired` - Auth issues
- `UnsupportedVersion` - Version incompatibilities
- `ESIntegrationException` - Base integration exception

Map these to Home Assistant exceptions:
- `ConfigEntryNotReady` - Temporary failures (retryable)
- `ConfigEntryAuthFailed` - Auth failures (requires reconfiguration)

### Logging

Use the custom logging module (`logger.py`):

```python
from custom_components.elasticsearch.logger import LOGGER, have_child

# Get a child logger for specific context
_logger = have_child(name="component_name")
_logger.info("Message with context")
```

Decorators are available:
- `@log_enter_exit_debug` - For sync functions
- `@async_log_enter_exit_debug` - For async functions (debug level)
- `@async_log_enter_exit_info` - For async functions (info level)

## Key Architectural Patterns

### Config Flow

The integration uses Home Assistant's config flow for setup:
- `config_flow.py` defines the UI flow
- Validates connectivity and credentials during setup
- Supports reconfiguration and options flow

### Event Publishing Pipeline

The core publishing logic is in `es_publish_pipeline.py`:
1. Events are collected from Home Assistant
2. Buffered for efficient batch processing
3. Published to Elasticsearch using the Bulk API
4. Handles retries and error scenarios

### Datastream Management

The integration supports multiple Elasticsearch features:
- Time Series Data Streams (TSDS)
- Datastream Lifecycle Management (DLM)
- Index Lifecycle Management (ILM)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [legrego/homeassistant-elasticsearch](https://github.com/legrego/homeassistant-elasticsearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
