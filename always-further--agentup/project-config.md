---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AgentUp is a Python framework for creating AI agents with production-ready features including security, scalability, and extensibility. It uses a configuration-driven architecture where agent behaviors, data sources, and workflows are defined through YAML configuration rather than code.

**Key Features:**
- Configuration-over-code approach with YAML-driven agent definitions
- Security-first design with scope-based access control and comprehensive audit logging
- Plugin ecosystem with community registry and automatic security scanning
- Multi-provider AI support (OpenAI, Anthropic, Ollama)
- MCP (Model Context Protocol) and A2A (Agent-to-Agent) protocol compliance
- Real-time operations with streaming, async processing, and push notifications

## Technology Stack

- **Python**: >=3.11 required
- **Pydantic**: v2 for data validation and settings management
- **Web Framework**: FastAPI with Uvicorn ASGI server
- **Package Manager**: UV (preferred) for dependency management
- **Plugin System**: Pluggy-based architecture with middleware inheritance
- **Authentication**: OAuth2, JWT, API key support via Authlib
- **Logging**: Structlog with correlation IDs for distributed tracing
- **Testing**: Pytest with async support and comprehensive markers
- **Code Quality**: Ruff (linting/formatting), MyPy (type checking), Bandit (security)

## Essential Development Commands

### Environment Setup
```bash
uv sync --all-extras --dev    # Install all dependencies including dev tools
uv pip install -e .           # Install package in editable mode
```

### Testing
```bash
# Unit tests only (fast)
uv run pytest tests/test_*.py tests/test_core/ tests/test_cli/ -v -m "not integration and not e2e and not performance"

# Integration tests
chmod +x tests/integration/int.sh && ./tests/integration/int.sh

# All tests with coverage
uv run pytest tests/ --cov=src --cov-report=html --cov-report=term-missing

# Watch mode for development
uv run pytest-watch --runner "uv run pytest tests/test_*.py tests/test_core/ tests/test_cli/ -m 'not integration and not e2e and not performance'"
```

### Code Quality (Required Before Commits)
```bash
uv run ruff check --fix src/ tests/    # Fix linting issues
uv run ruff format src/ tests/         # Format code
uv run mypy src/                       # Type checking
uv run bandit -r src/ -ll              # Security scanning
```

### Agent Development
```bash
uv run agentup init                    # Create new agent project
uv run agentup run                     # Start development server
uv run agentup validate                # Validate agent configuration
```

### Plugin Management
```bash
# Install plugins
uv add agentup-plugin-name             # Install plugin package
uv run agentup plugin add plugin-name  # Add to configuration

# Manage configuration
uv run agentup plugin sync             # Auto-sync installed plugins to config
uv run agentup plugin list             # List configured plugins
uv run agentup plugin remove plugin-name  # Remove from configuration
uv remove plugin-name                  # Uninstall plugin package

# Development
uv run agentup plugin reload plugin-name  # Reload plugin at runtime
uv run agentup plugin validate         # Validate plugin configuration
```

### Makefile Shortcuts
```bash
make install-dev      # Complete development setup
make test-unit        # Fast unit tests
make lint-fix         # Fix linting and formatting
make validate-all     # Run all quality checks
make clean            # Clean temporary files
```

## Code Architecture

### Core Structure
```
src/agent/
├── api/           # FastAPI server, routes, middleware
├── capabilities/  # Agent capability system and executors
├── cli/           # Command-line interface commands
├── config/        # Configuration models and loading
├── core/          # Function dispatching and execution
├── llm_providers/ # AI provider integrations
├── mcp_support/   # Model Context Protocol integration
├── plugins/       # Plugin system (pluggy-based)
├── security/      # Authentication, authorization, audit
├── services/      # Service layer abstractions
├── state/         # Conversation and state management
├── templates/     # Project generation templates
└── utils/         # Utility functions and helpers
```

### Key Architectural Patterns

**Plugin System**: Uses pluggy for hook-based architecture where plugins register capabilities with automatic middleware inheritance and scope-based permissions.

**Security Layer**: Unified authentication supporting multiple types (API key, JWT, OAuth2) with hierarchical scope-based authorization, comprehensive audit logging, and fail-secure design that denies access when security configuration is missing or invalid.

**Configuration-Driven**: Agent behavior defined through YAML files with Pydantic validation and environment variable overrides.

**Capability Registration**: AI functions are automatically discovered and registered with optional middleware (rate limiting, caching, retry logic) and state management.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [always-further/AgentUp](https://github.com/always-further/AgentUp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
