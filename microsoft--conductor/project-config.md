---
trigger: always_on
description: Conductor is a CLI tool for defining and running multi-agent workflows with the GitHub Copilot SDK. Workflows are defined in YAML and support parallel execution, conditional routing, loop-back patterns, and human-in-the-loop gates.
---

# AGENTS.md

## Project Overview

Conductor is a CLI tool for defining and running multi-agent workflows with the GitHub Copilot SDK. Workflows are defined in YAML and support parallel execution, conditional routing, loop-back patterns, and human-in-the-loop gates.

## Common Commands

```bash
# Install dependencies
make install          # or: uv sync
make dev              # install with dev dependencies

# Run tests
make test                                           # all tests
uv run pytest tests/test_engine/test_workflow.py   # single file
uv run pytest -k "test_parallel"                   # pattern match

# Run tests with coverage
make test-cov

# Lint and format
make lint             # check only
make format           # auto-fix and format

# Type check
make typecheck

# Run all checks (lint + typecheck)
make check

# Run a workflow
uv run conductor run workflow.yaml --input question="What is Python?"

# Run with web dashboard
uv run conductor run workflow.yaml --web --input question="What is Python?"

# Run in background (prints dashboard URL and exits)
uv run conductor run workflow.yaml --web-bg --input question="What is Python?"

# Stop a background workflow
uv run conductor stop                  # auto-stop if one running, list if multiple
uv run conductor stop --port 8080      # stop specific port
uv run conductor stop --all            # stop all background workflows

# Update conductor
uv run conductor update                # check for and install latest version

# Resume a failed workflow from checkpoint
uv run conductor resume workflow.yaml  # resume from latest checkpoint
uv run conductor checkpoints           # list available checkpoints

# Validate a workflow
uv run conductor validate examples/simple-qa.yaml
make validate-examples    # validate all examples
```

## Architecture

### Core Package Structure (`src/conductor/`)

- **cli/**: Typer-based CLI with commands `run`, `validate`, `init`, `templates`, `stop`, `update`, `resume`, `checkpoints`
  - `app.py` - Main entry point, defines the Typer application
  - `run.py` - Workflow execution command with verbose logging helpers
  - `bg_runner.py` - Background process forking for `--web-bg` mode
  - `pid.py` - PID file utilities for tracking/stopping background processes
  - `update.py` - Update check, version comparison, and self-upgrade via `uv tool install`

- **config/**: YAML loading and Pydantic schema validation
  - `schema.py` - Pydantic models for all workflow YAML structures (WorkflowConfig, AgentDef, ParallelGroup, ForEachDef, etc.)
  - `loader.py` - YAML parsing with environment variable resolution (${VAR:-default}) and `!file` tag support
  - `validator.py` - Cross-reference validation (agent names, routes, parallel groups)

- **engine/**: Workflow execution orchestration
  - `workflow.py` - Main `WorkflowEngine` class that orchestrates agent execution, parallel groups, for-each groups, and routing
  - `context.py` - `WorkflowContext` manages accumulated agent outputs with three modes: accumulate, last_only, explicit
  - `router.py` - Route evaluation with Jinja2 templates and simpleeval expressions
  - `limits.py` - Safety enforcement (max iterations, timeout)
  - `checkpoint.py` - Automatic checkpoint saving on failure and resume support

- **executor/**: Agent execution
  - `agent.py` - `AgentExecutor` handles prompt rendering, tool resolution, and output validation for single agents
  - `script.py` - `ScriptExecutor` runs shell commands as workflow steps, capturing stdout/stderr/exit_code
  - `template.py` - Jinja2 template rendering
  - `output.py` - JSON output parsing and schema validation

- **providers/**: SDK provider abstraction
  - `base.py` - `AgentProvider` ABC defining `execute()`, `validate_connection()`, `close()`
  - `copilot.py` - GitHub Copilot SDK implementation
  - `claude.py` - Anthropic Claude API implementation
  - `factory.py` - Provider instantiation

- **gates/**: Human-in-the-loop support
  - `human.py` - Rich terminal UI for human gate interactions

- **interrupt/**: Interactive workflow interruption (Esc/Ctrl+G to pause)
  - `listener.py` - Keyboard listener daemon thread for Esc/Ctrl+G detection

- **web/**: Real-time web dashboard for workflow visualization
  - `server.py` - FastAPI + uvicorn server with WebSocket broadcasting, late-joiner state replay, and `POST /api/stop` endpoint
  - `static/index.html` - Single-file Cytoscape.js frontend with DAG graph, agent detail panel, and streaming activity

- **events.py**: Pub/sub event system decoupling workflow execution from rendering (console, web dashboard)

- **exceptions.py**: Custom exception hierarchy (ConductorError, ValidationError, ExecutionError, etc.)

### Workflow Execution Flow

1. CLI parses YAML via `config/loader.py` → `WorkflowConfig`
2. `WorkflowEngine` initializes with config and provider
3. Engine loops: find agent/parallel/for-each/script → execute → evaluate routes → next
4. Parallel groups execute agents concurrently with context isolation (deep copy snapshot)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/conductor](https://github.com/microsoft/conductor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
