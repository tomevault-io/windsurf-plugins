---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Microsoft Agent 365 SDK for Python - Enterprise-grade extensions for building production-ready AI agents for M365, Teams, Copilot Studio, and Webchat. This is a multi-package monorepo organized as a uv workspace with 13 interdependent packages.

**Python Version**: 3.11+ (3.11 and 3.12 tested in CI)

**For detailed architecture and design documentation**, see [docs/design.md](docs/design.md), which includes:
- Complete package descriptions and usage examples
- Design patterns (Singleton, Context Manager, Builder, Result, Strategy)
- Data flow diagrams for agent invocation tracing and MCP tool discovery
- Configuration options and environment variables
- Per-package design documents in `libraries/<package-name>/docs/design.md`

## Development Commands

### Setup

```bash
# Create and activate virtual environment
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install dependencies (locks and syncs all workspace packages)
uv lock && uv sync --locked --all-extras --dev
```

### Building

```bash
# Build all packages (requires version environment variable)
export AGENT365_PYTHON_SDK_PACKAGE_VERSION="0.1.0"  # Windows: $env:AGENT365_PYTHON_SDK_PACKAGE_VERSION = "0.1.0"
uv build --all-packages --wheel

# Built wheels are output to dist/
```

### Testing

```bash
# Run all unit tests (excludes integration tests)
uv run --frozen pytest tests/ -v --tb=short -m "not integration"

# Run specific test file
uv run --frozen pytest tests/runtime/test_environment_utils.py -v

# Run tests matching pattern
uv run --frozen pytest tests/ -k "environment" -v

# Run integration tests (requires secrets/environment variables)
uv run --frozen pytest -m integration -v --tb=short

# Run with coverage report
pytest tests/ --cov=libraries --cov-report=html -v
```

**Test markers:**
- `unit`: Fast, mocked tests (default)
- `integration`: Slow tests requiring real services/API keys

### Running with tox

```bash
# Run all default environments (lint, format, unit tests on 3.11 + 3.12)
uv run tox

# Run a specific environment
uv run tox -e lint
uv run tox -e format
uv run tox -e py311
uv run tox -e py312

# Run integration tests (requires env vars)
uv run tox -e integration

# Verify centralized dependency constraints
uv run tox -e verify-constraints

# Pass extra args to pytest
uv run tox -e py311 -- -k "environment"

# List all available environments
uv run tox list
```

### Linting and Formatting

```bash
# Check linting (does not auto-fix)
uv run --frozen ruff check .

# Auto-fix linting issues
uv run --frozen ruff check . --fix

# Check formatting
uv run --frozen ruff format --check .

# Auto-format code
uv run --frozen ruff format .
```

**Linting rules:**
- Line length: 100 characters
- Enabled: pycodestyle (E/W), Pyflakes (F), isort (I), flake8-bugbear (B), comprehensions (C4), pyupgrade (UP), copyright headers (CPY)
- Copyright header required in all Python files (see Code Standards)

## Architecture

### Package Structure

The repository follows a **monorepo workspace** pattern with 13 packages organized into 4 core areas:

```
libraries/
├── Core Packages (foundation)
│   ├── microsoft-agents-a365-runtime           # Core utilities and extensions
│   ├── microsoft-agents-a365-notifications     # Notification services and models
│   ├── microsoft-agents-a365-observability-core    # OpenTelemetry-based tracing
│   └── microsoft-agents-a365-tooling           # Tool definitions and MCP integration
│
└── Framework Extensions (integrate with specific AI frameworks)
    ├── Observability Extensions
    │   ├── *-observability-extensions-openai
    │   ├── *-observability-extensions-langchain
    │   ├── *-observability-extensions-semantickernel
    │   └── *-observability-extensions-agentframework
    │
    └── Tooling Extensions
        ├── *-tooling-extensions-openai
        ├── *-tooling-extensions-semantickernel
        ├── *-tooling-extensions-agentframework
        └── *-tooling-extensions-azureaifoundry
```

### Key Architectural Patterns

1. **Namespace Packages**: All packages share the `microsoft_agents_a365` namespace
   - Directory names: `microsoft-agents-a365-*` (dashes)
   - Python imports: `microsoft_agents_a365.*` (underscores)

2. **Core + Extensions Pattern**:
   - Core packages (runtime, observability-core, tooling) provide framework-agnostic base functionality
   - Extension packages add framework-specific integrations (OpenAI, LangChain, Semantic Kernel, Agent Framework)
   - Extensions depend on core packages but cores are independent

3. **Workspace Dependencies**:
   - Inter-package dependencies managed via `[tool.uv.workspace]` in root pyproject.toml
   - Packages reference each other using `{ workspace = true }`
   - All packages built and versioned together

4. **Observability**:
   - Built on OpenTelemetry (traces, spans, metrics)
   - Core provides base functionality; extensions add framework-specific instrumentation
   - Azure Monitor and Jaeger export options available

5. **Tooling**:
   - Tool definitions for agent capabilities
   - MCP (Model Context Protocol) integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/Agent365-python](https://github.com/microsoft/Agent365-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
