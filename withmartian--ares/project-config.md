---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ARES (Agentic Research and Evaluation Suite) is an RL-first framework for training and evaluating code agents. It implements an async version of DeepMind's dm_env specification, treating LLM requests as observations and LLM responses as actions within a standard RL loop.

## Development Commands

### Setup
```bash
# Install all dependencies including dev tools
uv sync --all-groups

# Install only main dependencies
uv sync

# Install with specific groups
uv sync --group dev
uv sync --group examples
```

### Testing
```bash
# Run all tests
uv run pytest

# Run a specific test file
uv run pytest src/ares/config_test.py

# Run tests matching a pattern
uv run pytest -k "test_pattern_name"
```

Unit tests follow the `*_test.py` naming pattern (preferred) or `test_*.py` and are colocated with source files in `src/`. Integration and end-to-end tests may live under `integration_tests/` at the root.

### Code Quality
```bash
# Lint (check for issues)
uv run ruff check

# Format code
uv run ruff format

# Type checking
uv run pyright

# Fix auto-fixable linting issues
uv run ruff check --fix
```

The project uses ruff for linting/formatting (line length: 120) and follows Google-style docstrings.

### Running Examples
```bash
# Sequential eval with local Docker + local LLM
uv run -m examples.01_sequential_eval_with_local_llm

# Sequential eval with API
uv run -m examples.02_sequential_eval_with_api

# Parallel eval with API
uv run -m examples.03_parallel_eval_with_api
```

Examples demonstrate basic usage patterns and require the examples dependency group.

## Public API

The primary user-facing API is the registry system exposed at the top level:

```python
import ares

# Create environment from preset (defaults to local Docker containers)
env = ares.make("sbv-mswea")

# Task selection syntax
env = ares.make("sbv-mswea:0")       # Single task at index 0
env = ares.make("sbv-mswea:0:10")    # Slice of tasks 0-9
env = ares.make("sbv-mswea@2/8")     # Shard 2 of 8

# List available presets
ares.info()                           # All presets
ares.info("sbv-mswea")               # Specific preset info

# Override container backend
from ares.containers import daytona
env = ares.make("sbv-mswea", container_factory=daytona.DaytonaContainer)
```

The registry is populated by `presets.py` (imported automatically by `__init__.py`). Custom presets can be registered via `ares.registry.register_preset()` or the `@register_env` decorator.

## High-Level Architecture

### Core Abstraction: The RL Loop

ARES treats code agent interactions as a reinforcement learning problem:

1. **Environment** emits an **Observation** (LLM request with task context)
2. **Agent** receives observation and returns an **Action** (LLM response with code/commands)
3. **Environment** processes action, executes commands in container, returns next observation
4. Loop continues until episode terminates (success, step limit, or explicit submission)
5. **Reward** is computed at episode end (e.g., 1.0 if tests pass, 0.0 otherwise)

### Key Components

#### 1. Environments (`src/ares/environments/`)

**Base (`base.py`):**
- `Environment` protocol defines the dm_env interface (`reset()`, `step()`, `close()`)
- `TimeStep` namedtuple for observations, rewards, and episode signals
- `create_container()` helper for creating containers from images or Dockerfiles
- `Janitor` class for emergency cleanup of containers on abnormal termination

**Implementation (`code_env.py`):**
- `CodeEnvironment` - Concrete environment for Harbor-compatible datasets (including SWE-bench)
- Orchestrates the entire RL loop: manages container lifecycle, code agent execution, and LLM request interception
- Builds containers from Dockerfiles with configurable resources (CPU, memory, disk)
- Reads reward from `/reward.txt` or `/reward.json` in the container
- Uses async context manager pattern (`async with env:`) for guaranteed cleanup

**Episode Termination:**
- Step limit reached (default: 250 steps for CodeEnvironment)
- Agent explicitly submits (signals completion)
- Container or agent error

#### 2. Code Agents (`src/ares/code_agents/`)

**Interface:**
```python
class CodeAgent(Protocol):
    async def run(self, task: str) -> None
```

**Main Implementation:**
- `MiniSWECodeAgent` (`mini_swe_agent.py`) - Wraps the mini-swe-agent library
  - Uses Jinja2-rendered system/instance templates
  - Parses bash commands from markdown code blocks
  - Handles format errors, timeouts, and submission signals
  - Tracks performance statistics via `StatTracker`

**Factory Pattern:**
- `CodeAgentFactory[T]` protocol creates agents with container and LLM client dependencies
- Enables dependency injection and easy agent swapping

#### 3. Containers (`src/ares/containers/`)

**Abstract Protocol (`containers.py`):**
```python
class Container(Protocol):
    async def start(env: dict[str, str] | None) -> None
    async def exec_run(command, workdir, env, timeout_s) -> ExecResult
    async def upload_files/download_files/upload_dir/download_dir
    def stop_and_remove() -> None  # Sync for atexit cleanup
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/withmartian) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
