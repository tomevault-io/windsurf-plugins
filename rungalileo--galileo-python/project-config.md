---
trigger: always_on
description: Galileo Python SDK (`galileo` on PyPI) - the official Python client library for the Galileo AI platform. It enables logging and tracing of LLM calls, experiments, datasets, prompt management, and more.
---

## Project Overview

Galileo Python SDK (`galileo` on PyPI) - the official Python client library for the Galileo AI platform. It enables logging and tracing of LLM calls, experiments, datasets, prompt management, and more.

**Key characteristics:**
- Public SDK published to PyPI (external contributors welcome)
- Depends on `galileo-core` for shared schemas and infrastructure (see Known Issues)
- Uses auto-generated API client from OpenAPI specification
- Supports multiple LLM frameworks: OpenAI, LangChain, CrewAI, OpenAI Agents SDK

## Build & Development Commands

```bash
# Install dependencies (requires poetry)
poetry install --all-extras --no-root

# Full setup (install + pre-commit hooks)
inv setup

# Run all tests (parallel by default)
poetry run pytest

# Run single test file
poetry run pytest tests/test_decorator.py

# Run single test
poetry run pytest tests/test_decorator.py::test_function_name -v

# Run tests with coverage
inv test

# Type checking
inv type-check

# Linting (via pre-commit)
poetry run ruff check --fix src/
poetry run ruff format src/
```

## Architecture

### Package Structure

```
src/galileo/
├── __future__/              # New object-centric API (WIP)
│   ├── project.py           # Project domain object
│   ├── dataset.py           # Dataset domain object
│   ├── experiment.py        # Experiment domain object
│   ├── prompt.py            # Prompt domain object
│   ├── log_stream.py        # LogStream domain object
│   ├── configuration.py     # Configuration management
│   └── shared/              # Shared utilities (filters, sorting, base classes)
├── logger/                  # Core logging functionality
│   └── logger.py            # GalileoLogger - central trace/span management
├── handlers/                # Framework-specific integrations
│   ├── langchain/           # LangChain callback handler (GalileoCallback)
│   ├── crewai/              # CrewAI event listener
│   └── openai_agents/       # OpenAI Agents SDK integration
├── openai/                  # Drop-in OpenAI client wrapper (auto-logging)
├── resources/               # Auto-generated API client (DO NOT EDIT)
├── schema/                  # Pydantic models for SDK-specific types
├── utils/                   # Utility functions and helpers
├── datasets.py              # Dataset service (current API)
├── experiments.py           # Experiment service (current API)
├── prompts.py               # Prompt service (current API)
├── projects.py              # Project service (current API)
├── log_streams.py           # LogStream service (current API)
├── decorator.py             # @log decorator and galileo_context
└── config.py                # GalileoPythonConfig configuration
```

### Core Components

**GalileoLogger** (`src/galileo/logger/logger.py`): Central class for uploading traces to Galileo. Supports batch and streaming modes. Manages traces, spans (LLM, retriever, tool, workflow, agent), and sessions.

**Decorators** (`src/galileo/decorator.py`): The `@log` decorator and `galileo_context` context manager for automatic function tracing. Uses ContextVars for thread-safe nested span tracking.

**Handlers** (`src/galileo/handlers/`): Framework-specific integrations:
- `langchain/` - LangChain callback handler (`GalileoCallback`)
- `crewai/` - CrewAI handler (uses lazy imports to avoid side effects)
- `openai_agents/` - OpenAI Agents SDK integration

**OpenAI Wrapper** (`src/galileo/openai/`): Drop-in replacement for OpenAI client that auto-logs calls.

**`__future__` Package** (`src/galileo/__future__/`): New object-centric API implementing the "Golden Flow" patterns. Provides intuitive, Pythonic interfaces for domain objects (Project, Dataset, Prompt, Experiment, LogStream). Released incrementally as stable.

### Auto-Generated Code

**Resources** (`src/galileo/resources/`): Auto-generated API client from OpenAPI spec. **Excluded from linting/type-checking.** Never edit manually.

```bash
# Regenerate API client
./scripts/import-openapi-yaml.sh https://api.galileo.ai/client
./scripts/auto-generate-api-client.sh
```

**Important:** The OpenAPI spec comes from the **Client API** (`/client`), not the main API (`/docs`). The Client API is a curated subset designed specifically for SDK consumption.

### Dependency on galileo-core

The SDK depends on `galileo-core` for shared schemas, helpers, and base classes:
- `galileo_core.schemas.logging.*` - Span types (LlmSpan, ToolSpan, etc.), Trace, Session
- `galileo_core.helpers.*` - API key management, execution utilities
- `galileo_core.schemas.protect.*` - Protection/guardrails schemas

**Note:** There is ongoing work to reduce/eliminate this dependency. See Known Issues section.

## Key Patterns

### Object-Centric Design (`__future__` package)

Domain objects follow consistent patterns:

```python
from galileo.__future__ import Project, Dataset

# Factory methods (class-level)
project = Project.get(name="my-project")      # Retrieve existing
projects = Project.list()                      # List all

# Instance creation with lifecycle
project = Project(name="new-project")          # LOCAL_ONLY state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rungalileo/galileo-python](https://github.com/rungalileo/galileo-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
