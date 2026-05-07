---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a workbench designed for security research to test attacks against large language models and prompt injection defenses. The project uses `pydantic-ai` as its core AI framework and supports multiple environments for testing. It features a Hydra-based configuration system for flexible experiment orchestration with parameter sweeps and overrides.

## Development Commands

### Linting and Type Checking
```sh
uv run ruff check --fix
uv run ruff format
uv run ty check
```

### Testing
```sh
# Run all tests (unit + integration)
uv run pytest -vx

# Run unit tests only (fast, no Docker needed - CI/CD default)
uv run pytest -vx -m "not docker_integration"

# Run integration tests only (requires Docker)
uv run pytest -vx -m docker_integration
```

**Note**: Integration tests require Docker to be running. CI/CD pipelines skip integration tests by default for speed.

### Running Experiments
```sh
# Export default configuration (first time setup)
uv run prompt-siren config export

# Run benign-only evaluation
uv run prompt-siren run benign +dataset=agentdojo-workspace

# Run attack evaluation
uv run prompt-siren run attack +dataset=agentdojo-workspace +attack=template_string

# Run SWE-bench evaluation
uv run prompt-siren run benign +dataset=swebench

# Override parameters
uv run prompt-siren run benign +dataset=agentdojo-workspace agent.config.model=azure:gpt-5 execution.concurrency=4

# Parameter sweep (use hydra.mode=MULTIRUN for sweeping over multiple values)
uv run prompt-siren run benign --multirun +dataset=agentdojo-workspace agent.config.model=azure:gpt-5,azure:gpt-5-nano

# Parameter sweep with multiple parameters
uv run prompt-siren run benign --multirun +dataset=agentdojo-workspace agent.config.model=azure:gpt-5,azure:gpt-5-nano execution.concurrency=1,4

# Parameter sweep with attacks
uv run prompt-siren run attack --multirun +dataset=agentdojo-workspace +attack=template_string,mini-goat

# Validate configuration
uv run prompt-siren config validate +dataset=agentdojo-workspace +attack=template_string

# Run with config file that includes dataset/attack (no overrides needed)
uv run prompt-siren run attack --config-dir=./my_config
```

**Note**: Dataset and attack can be specified either via command-line overrides (`+dataset=...`, `+attack=...`) or by including them in your config file's `defaults` list. See docs/configuration.md for details.

**Platform Requirements**:
- Linux or macOS only (Windows not supported)
- Docker required for SWE-bench
- Base Docker images must have `/bin/bash` available

### Development guidelines

- Always use modern 3.10+ type hints:
  - `list`, `dict`, `set` instead of importing from `typing`.
  - `|` instead of `Union` and `Optional`.
- Always write meaningful tests for new features. Avoid tests that are obvious.
- Always lint and type check.
- Avoid using `type: ignore`, unless the issue is the result of a deliberate choice (e.g., in tests).
- Avoid using `cast`.
- Only use module-level imports instead of local imports, unless it's to import optional dependencies or to handle circular imports.

## Architecture

### Core Components

1. **Agent System** (`src/prompt_siren/agents/`)
   - `abstract.py` - Defines the `AbstractAgent` protocol for agent interfaces
   - `plain.py` - Implements the main agent logic for running tasks with pydantic-ai
   - `_utils.py` - Helper utilities for agent operations
   - `registry.py` - Agent plugin registration system
   - Supports tool execution with injection attack capabilities

2. **Dataset System** (`src/prompt_siren/datasets/`)
   - `abstract.py` - Defines `AbstractDataset` protocol for dataset interfaces
   - `agentdojo_dataset.py` - AgentDojo dataset implementation
   - `swebench_dataset/` - SWE-bench dataset for code editing benchmarks
     - Uses multi-stage Docker builds (base/env/instance) with caching
     - Jinja2 template system for prompts
     - Official SWE-bench test harness for evaluation
   - `registry.py` - Dataset plugin registration system
   - Datasets provide:
     - Collections of tasks (benign, malicious, and task couples)
     - Specification of which environment type to use
     - Environment configuration for that environment
   - Separates task data from execution context (environments)
   - Allows multiple datasets to share the same environment type

3. **Environment System** (`src/prompt_siren/environments/`)
   - `abstract.py` - Base `Environment` class provides the interface for different execution environments
   - Environment states (`env_state`) are managed per-task and passed separately from environment
   - Two-level context management for resource lifecycle:
     - `create_batch_context()` - Batch-level setup for expensive resources (browsers, servers)
     - `create_task_context()` - Per-task context with fresh environment state
   - Environments handle rendering of outputs and injection vector detection
   - `registry.py` - Environment plugin registration system
   - Key environments:
     - `agentdojo/` - AgentDojo environment integration
     - `playwright.py` - Playwright-based web automation environment


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [facebookresearch/prompt-siren](https://github.com/facebookresearch/prompt-siren) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
