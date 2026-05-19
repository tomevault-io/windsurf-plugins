---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the **Microsoft 365 Agents SDK for Python**, a framework for building enterprise-grade conversational agents for M365, Teams, Copilot Studio, and other platforms. The SDK replaces the legacy Bot Framework SDK (botbuilder packages) with a modern, modular architecture.

**Important**: Python imports use underscores (`microsoft_agents`), not dots (`microsoft.agents`).

## Development Setup

### Initial Setup

**Quick setup (Linux/macOS)**:
```bash
. ./scripts/dev_setup.sh
```

**Quick setup (Windows)**:
```bash
. ./scripts/dev_setup.ps1
```

**Manual setup** (from repository root):
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install all libraries in editable mode
pip install -e ./libraries/microsoft-agents-activity/ --config-settings editable_mode=compat
pip install -e ./libraries/microsoft-agents-hosting-core/ --config-settings editable_mode=compat
pip install -e ./libraries/microsoft-agents-authentication-msal/ --config-settings editable_mode=compat
pip install -e ./libraries/microsoft-agents-copilotstudio-client/ --config-settings editable_mode=compat
pip install -e ./libraries/microsoft-agents-hosting-aiohttp/ --config-settings editable_mode=compat
pip install -e ./libraries/microsoft-agents-hosting-teams/ --config-settings editable_mode=compat
pip install -e ./libraries/microsoft-agents-storage-blob/ --config-settings editable_mode=compat
pip install -e ./libraries/microsoft-agents-storage-cosmos/ --config-settings editable_mode=compat
pip install -e ./libraries/microsoft-agents-hosting-fastapi/ --config-settings editable_mode=compat

# Install development dependencies
pip install -r dev_dependencies.txt

# Setup pre-commit hooks
pre-commit install
```

**Python version**: Requires Python 3.10+, supports 3.10-3.14. Recommended: Python 3.11+

## Common Commands

### Testing

```bash
# Run all tests
pytest

# Run tests in a specific directory
pytest tests/microsoft-agents-activity/

# Run a single test file
pytest tests/microsoft-agents-activity/test_activity.py

# Run a single test
pytest tests/microsoft-agents-activity/test_activity.py::test_activity_creation

# Run with verbose output
pytest -v

# Run with test markers
pytest -m unit
pytest -m integration
pytest -m slow
```

### Code Quality

```bash
# Format code with black (line length: 88)
black libraries/

# Check formatting without making changes
black libraries/ --check

# Lint with flake8 (max line length: 127, max complexity: 10)
flake8 .

# Run pre-commit checks manually
pre-commit run --all-files
```

### Building Packages

```bash
# Set package version (from versioning directory)
cd ./versioning
setuptools-git-versioning

# Build all packages (run from repository root)
mkdir -p dist
for dir in libraries/*; do
  if [ -f "$dir/pyproject.toml" ]; then
    (cd "$dir" && python -m build --outdir ../../dist)
  fi
done

# Build a specific package
cd libraries/microsoft-agents-activity
python -m build
```

## Architecture Overview

### Layer Structure

The SDK follows a **layered, modular architecture** with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────────┐
│ Layer 5: Web Framework Adapters                                 │
│ hosting-aiohttp, hosting-fastapi                                │
├─────────────────────────────────────────────────────────────────┤
│ Layer 4: Platform Extensions & Storage                          │
│ hosting-teams, storage-blob, storage-cosmos                     │
├─────────────────────────────────────────────────────────────────┤
│ Layer 3: Authentication                                          │
│ authentication-msal                                              │
├─────────────────────────────────────────────────────────────────┤
│ Layer 2: Core Hosting Engine                                    │
│ hosting-core (Agent, TurnContext, State, Routing)              │
├─────────────────────────────────────────────────────────────────┤
│ Layer 1: Protocol/Schema                                        │
│ activity (Activity protocol, Pydantic models)                   │
└─────────────────────────────────────────────────────────────────┘
```

### Package Dependencies

Each library in `libraries/` is independently published to PyPI:

| Package | Purpose | Key Abstractions |
|---------|---------|------------------|
| `microsoft-agents-activity` | Activity protocol types using Pydantic | `Activity`, `ConversationReference`, protocols |
| `microsoft-agents-hosting-core` | Core agent runtime and lifecycle | `Agent`, `TurnContext`, `ActivityHandler`, `AgentApplication` |
| `microsoft-agents-authentication-msal` | MSAL-based OAuth authentication | `MsalAuth`, `MsalConnectionManager` |
| `microsoft-agents-hosting-aiohttp` | aiohttp web framework adapter | `CloudAdapter`, `start_agent_process()` |
| `microsoft-agents-hosting-fastapi` | FastAPI web framework adapter | `CloudAdapter`, `start_agent_process()` |
| `microsoft-agents-hosting-teams` | Teams-specific extensions | `TeamsActivityHandler`, `TeamsInfo` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/Agents-for-python](https://github.com/microsoft/Agents-for-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
