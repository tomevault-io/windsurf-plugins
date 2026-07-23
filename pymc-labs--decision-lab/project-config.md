---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

dlab is a Python CLI that runs [opencode](https://opencode.ai) in automated mode, sandboxed with Docker, with built-in parallel subagent capabilities. The CLI runs locally and orchestrates Docker containers running opencode via `docker exec`.

## Development Environment

- Python 3.10+, Docker required
- Virtual environment: Always use `venv/` (not `.venv/`)
- Activate: `source venv/bin/activate`
- Install dev dependencies: `pip install -e ".[dev]"`

### Running Tests

```bash
# All tests
~/miniconda3/envs/dlab-testing/bin/python -m pytest tests/ -v

# Single test file
~/miniconda3/envs/dlab-testing/bin/python -m pytest tests/test_config.py -v

# Single test
~/miniconda3/envs/dlab-testing/bin/python -m pytest tests/test_config.py::test_function_name -v
```

- NEVER use mocks - tests use real implementations
- NEVER skip tests - report failures and ask user whether to ignore or start resources
- Do NOT set `PYTESTER_FLAGS="cxx="` - C++ compilation is faster for PyTensor deterministics

## Running the CLI

**IMPORTANT: This is how you run the CLI. Do not forget.**

```bash
# Basic usage
dlab --dpack <dpack> --data <data-dir> --env-file .env --work-dir <work-dir> --prompt "Your prompt here"

# Multiple data files
dlab --dpack <dpack> --data file1.csv file2.csv --prompt "Compare these"

# Resume interrupted session
dlab --dpack <dpack> --continue-dir ./dlab-mmm-workdir-001 --prompt "Continue"

# Prompt from file
dlab --dpack <dpack> --data <data-dir> --prompt-file prompt.txt
```

Required flags:
- `--dpack` - Path to decision-pack config directory
- `--data` - Data files or directory (optional if decision-pack sets `requires_data: false`)
- `--env-file` - Environment file with API keys (auto-detected from decision-pack `.env`)
- `--prompt` or `--prompt-file` - Prompt text or file (optional if decision-pack sets `requires_prompt: false`)

Optional flags:
- `--model` - Override default model from config
- `--work-dir` - Explicit work directory path
- `--continue-dir` - Resume from a previous session's work directory
- `--rebuild` - Force rebuild Docker image

Environment variables starting with `DLAB_` are automatically forwarded from the host to the Docker container. decision-packs can use these for configuration (e.g., `DLAB_FIT_MODEL_LOCALLY=1` in the MMM decision-pack).

### Subcommands

```bash
# Interactive wizard to create a new decision-pack
dlab create-dpack [OUTPUT_DIR]

# Interactive wizard to create a parallel agent config
dlab create-parallel-agent [DPACK_DIR]

# Install decision-pack as a shortcut command
dlab install <dpack-path> [--bin-dir PATH]

# Live-monitor a session (TUI)
dlab connect <work-dir> [--log] [--log-json]

# View execution timeline with Gantt chart
dlab timeline [work-dir]

# Browser-based session viewer (DAG visualization)
dlab view <work-dir> [--port PORT] [--no-open]
```

## Coding Style

### Formatting & Organization
- Manual PEP8 formatting (no Black/Ruff)
- Imports: Always at top of file, sorted with isort (stdlib, third-party, local)
- File organization: Constants -> Functions & Classes (grouped logically) -> main
- File naming: snake_case (my_module.py)
- String formatting: f-strings preferred

### Type Annotations
- Full type annotations on all functions and variables

### Documentation
- NumPy-style docstrings with Parameters/Returns sections

### Error Handling
- Use built-in exceptions (ValueError, TypeError, etc.) with descriptive messages
- NEVER catch errors for functions that have their own error handling
- NEVER wrap code in generic `try/except Exception: pass` or `except: pass`
- Minimal logging: only errors and critical info

### Patterns to AVOID
- Global mutable state, singletons
- Over-abstraction (unnecessary base classes/interfaces)
- Classes where simple functions suffice (project is function-heavy)
- Imports anywhere except top of file

### Configuration
- Plain dictionaries loaded from YAML (config.yaml)
- No pydantic or dataclasses for config

### Agent System Prompts (in opencode/agents/*.md)
- NEVER put hard-coded values in agent system prompts
- Use placeholder syntax like `<VALUE>`, `<CHANNEL_COLS>`, `<DATE_COLUMN>` instead
- Keep examples minimal with placeholders - prompts are templates, not implementations

## Project Architecture

- decision-pack config contains: `docker/`, `opencode/`, `config.yaml`
- CLI orchestrates Docker containers running opencode
- Communication via `docker exec`
- Session state in JSON files for resumption
- Containers automatically stopped on completion, error, or interrupt (SIGINT/SIGTERM)
- Work directories are auto-numbered: `dlab-{dpack}-workdir-001`, `dlab-{dpack}-workdir-002`, etc.

## Modules (in `dlab/`)

- `cli.py` - CLI with argparse, subcommands, Rich output
- `config.py` - decision-pack config loading and validation
- `session.py` - Session creation and state management
- `docker.py` - Docker image building and container lifecycle
- `parallel_tool.py` - Loads parallel-agents.ts from `js/`
- `js/parallel-agents.ts` - TypeScript source bundled as package data
- `timeline.py` - Log parsing and timeline visualization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pymc-labs/decision-lab](https://github.com/pymc-labs/decision-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
