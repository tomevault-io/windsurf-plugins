---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

hexDAG is an **operating system for AI agents** -- an orchestration framework that provides pipelines (processes), ports (syscalls), drivers, and a standard library so that AI agents don't reinvent orchestration. It transforms complex AI workflows into deterministic, testable, and maintainable systems through declarative YAML configurations and DAG-based execution.

## Development Commands

### Environment Setup
```bash
# Install dependencies using uv (Python package manager)
uv sync

# Install with notebook support (for documentation)
uv sync --all-extras

# Install pre-commit hooks
uv run pre-commit install
```

### Notebooks
```bash
# Start Jupyter for interactive development
jupyter notebook notebooks/

# Execute and validate all notebooks
uv run python scripts/check_notebooks.py

# Format notebooks
uv run nbqa ruff notebooks/ --fix
uv run nbqa pyupgrade notebooks/ --py312-plus

# Strip notebook outputs (automatic via pre-commit)
uv run nbstripout notebooks/**/*.ipynb
```

### Testing
```bash
# Run all tests
uv run pytest

# Run tests with coverage
uv run pytest --cov=hexdag --cov-report=html --cov-report=term-missing

# Run specific test areas
uv run pytest tests/hexdag/kernel/pipeline_builder/ -x --tb=short  # Pipeline builder tests
uv run pytest tests/hexdag/kernel/                                 # Kernel tests
uv run pytest tests/hexdag/stdlib/lib/                             # System lib tests

# Run doctests (tests embedded in docstrings)
uv run pytest --doctest-modules hexdag/ --ignore=hexdag/cli/
uv run pytest --doctest-modules hexdag/ --ignore=hexdag/cli/ --doctest-continue-on-failure  # See all failures
```

### Code Quality
```bash
# Run all pre-commit hooks
uv run pre-commit run --all-files

# Linting and formatting
uv run ruff check hexdag/ --fix         # Linting with auto-fix
uv run ruff format hexdag/              # Code formatting
uv run mypy hexdag/                     # Type checking
uv run pyright hexdag/                  # Alternative type checker
uv run bandit -r hexdag                 # Security scanning

# Dependency analysis
uv run deptry .                        # Unused dependencies
uv run safety check                    # Vulnerability scanning

# Code quality metrics
uv run vulture hexdag/ --min-confidence 90    # Dead code detection
uv run radon cc hexdag/ --min B               # Complexity analysis

# Coverage and testing
uv run pytest --cov=hexdag --cov-report=html --cov-report=term-missing
```

### Examples
```bash
# Run specific examples
uv run examples/api_call_node_example.py         # API call nodes
uv run examples/demo/run_demo_pitch.py           # Demo startup pitch
uv run examples/libs/run_order_lifecycle.py       # Entity lifecycle
uv run examples/libs/run_database_agent.py        # Database agent tools
```

### Utilities
```bash
# Check test structure consistency
uv run scripts/check_test_structure.py

# Check examples functionality
uv run scripts/check_examples.py
```

## Architecture Overview

hexDAG is structured like an operating system -- kernel (execution engine), stdlib (built-in components), drivers (infrastructure), and api (user-facing tools):

### Framework Structure
```
hexdag/
├── kernel/                  # Core execution engine (/kernel)
│   ├── domain/              #   Domain models (DAG, NodeSpec, PipelineRun, etc.)
│   ├── orchestration/       #   Orchestrator, events, observers
│   ├── pipeline_builder/    #   YAML pipeline building and compilation
│   ├── ports/               #   Port interfaces (LLM, DataStore, PipelineSpawner)
│   ├── validation/          #   Type validation and schema conversion
│   ├── service.py           #   Service base class + @tool/@step decorators
│   └── lib_base.py          #   HexDAGLib base class (DEPRECATED → Service)
├── stdlib/                  # Standard library (/lib)
│   ├── adapters/            #   Built-in adapters (OpenAI, SQLite, Mock, etc.)
│   ├── nodes/               #   Node factories (LLMNode, AgentNode, etc.)
│   ├── macros/              #   Macro components (ReasoningAgent, etc.)
│   ├── prompts/             #   Prompt templates (tool prompts, etc.)
│   └── lib/                 #   System libs (ProcessRegistry, EntityState, Scheduler)
├── drivers/                 # Low-level infrastructure (/drivers)
│   ├── executors/           #   LocalExecutor (Executor)
│   ├── observer_manager/    #   LocalObserverManager (ObserverManager)
│   └── pipeline_spawner/    #   LocalPipelineSpawner (PipelineSpawner)
├── api/                     # Unified API layer (/usr/bin)
│   ├── execution.py         #   Pipeline execution
│   ├── processes.py         #   Process management (9 MCP tools)
│   └── ...                  #   Components, validation, documentation
├── docs/                    # Documentation utilities
└── cli/                     # Command-line interface
```

### Uniform Entity Pattern

All framework entities follow one pattern: **kernel defines contract, stdlib ships builtins, users write their own.**

| Entity   | Contract (kernel/)          | Builtins (stdlib/)                 | User custom        |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omniviser/hexDAG](https://github.com/omniviser/hexDAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
