---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

The `anomaly-agent` package is a Python library for detecting anomalies in time series data using Large Language Models. The architecture is built around a few key components:

### Core Components

- **AnomalyAgent** (`anomaly_agent/agent.py`): Enhanced main agent class with modern LangGraph patterns, Pydantic-based configuration, and robust error handling
- **AgentConfig**: Validated configuration management with built-in constraints and type safety
- **AgentState**: Enhanced Pydantic state model with validation, error tracking, and processing metadata
- **Detection/Verification Pipeline**: Modern LangGraph implementation with proper routing, error handling, and retry mechanisms
- **Pydantic Models**: Comprehensive structured models (`Anomaly`, `AnomalyList`, `AgentConfig`, `AgentState`) with v2 field validators
- **Prompt System** (`anomaly_agent/prompt.py`): Advanced customizable prompts with improved statistical criteria and domain awareness

### Key Files

- `anomaly_agent/agent.py`: Main agent implementation with LangGraph state machine
- `anomaly_agent/utils.py`: Utility functions for data generation and anomaly configuration
- `anomaly_agent/plot.py`: Plotting utilities for visualizing time series and anomalies
- `anomaly_agent/constants.py`: Configuration constants and default values
- `tests/`: Comprehensive test suite with architecture, agent behavior, and prompt functionality tests
  - `test_agent.py`: Core agent functionality and backward compatibility
  - `test_prompts.py`: Prompt system validation and customization
  - `test_graph_architecture.py`: Modern architecture features (GraphManager, class-based nodes, caching)

## Development Commands

**This project now uses `uv` for fast, reliable dependency management!** All commands automatically manage the virtual environment (.venv) using uv.

**Key uv benefits:**
- ⚡ **Faster installs**: 10-100x faster than pip
- 🔒 **Reproducible builds**: `uv.lock` ensures consistent dependency versions
- 🚀 **Automatic venv management**: No need to manually create/activate virtual environments
- 📦 **Modern dependency resolution**: Better conflict resolution and version selection
- 🔄 **Backward compatibility**: All existing make commands continue to work

### Environment Setup
```bash
# Everything is automatic with uv! Just sync dependencies:
make sync          # Install runtime dependencies only
make sync-dev      # Install runtime + development dependencies
make install-dev   # Alias for sync-dev

# Legacy aliases (still work for backward compatibility):
make requirements-install  # Maps to: uv sync
make requirements-dev      # Maps to: uv sync --group dev
```

### Testing
```bash
# Run all tests with coverage (uv manages .venv automatically)
make test
# or
make tests

# For specific test files:
uv run pytest tests/test_agent.py -v                      # Core agent functionality
uv run pytest tests/test_prompts.py -v                    # Prompt system tests
uv run pytest tests/test_graph_architecture.py -v         # Advanced architecture tests
uv run pytest tests/test_streaming_parallel.py -v         # Streaming and parallel features

# Run architecture-specific tests:
uv run pytest tests/test_graph_architecture.py::TestGraphManager -v           # Graph caching tests
uv run pytest tests/test_graph_architecture.py::TestDetectionNode -v          # Class-based node tests
uv run pytest tests/test_graph_architecture.py::TestErrorHandlerNode -v       # Error handling tests

# Integration tests (requires OPENAI_API_KEY in .env - automatically loaded by AnomalyAgent)
uv run pytest tests/ -m integration -v
```

### Code Quality
```bash
# Install pre-commit hooks (uv manages dependencies automatically)
make pre-commit-install

# Run all pre-commit checks
make pre-commit

# Auto-fix formatting issues
make pre-commit-fix

# Individual tools using uv:
uv run black anomaly_agent/  # Code formatting (line-length: 88)
uv run isort anomaly_agent/  # Import sorting
uv run flake8 anomaly_agent/ # Linting
uv run mypy anomaly_agent/   # Type checking
```

### Dependencies
```bash
# Install dependencies with uv
make sync-dev       # Install all dependencies (runtime + dev)
make sync           # Install runtime dependencies only

# Add new dependencies:
make add PACKAGE=<package-name>              # Add runtime dependency
make add-dev PACKAGE=<package-name>          # Add development dependency

# Remove dependencies:
make remove PACKAGE=<package-name>           # Remove any dependency

# Update all dependencies:
make update                                  # Equivalent to: uv sync --upgrade

# Lock dependencies for reproducible builds:
make lock                                    # Create/update uv.lock
```

### uv-Specific Commands
```bash
# Direct uv commands (for advanced usage):
uv add pandas                    # Add runtime dependency
uv add --group dev pytest        # Add development dependency
uv remove matplotlib             # Remove dependency
uv sync                          # Install dependencies from pyproject.toml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andrewm4894/anomaly-agent](https://github.com/andrewm4894/anomaly-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
