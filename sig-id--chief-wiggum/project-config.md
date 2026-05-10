---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chief Wiggum is an autonomous task runner that orchestrates workers to execute software engineering tasks using Claude Code. It parses tasks from a Kanban board (`.ralph/kanban.md`), spawns isolated workers via git worktrees, runs multi-stage pipelines of specialized agents, and creates pull requests.

**Language**: Bash 4.0+ with Python TUI component

## Commands

### Running Tests
```bash
./tests/test-runner.sh                      # Run all Bash tests
./tests/test-runner.sh --filter <pattern>   # Run filtered tests
./tests/run-all-tests.sh                    # Run ALL tests (Bash + TUI)
./tests/run-all-tests.sh --quick            # Quick mode (skip slow tests)
./tests/tui-test-runner.sh                  # Run TUI tests only
./tests/run-coverage.sh                     # Code coverage
./tests/run-shellcheck.sh                   # Bash linting
bash tests/test_<name>.sh                   # Run specific test file
```

### Development
```bash
export WIGGUM_HOME=$(pwd)
export PATH="$WIGGUM_HOME/bin:$PATH"
```

### Debugging
```bash
wiggum inspect worker TASK-001              # Inspect worker state
wiggum inspect pipeline                     # Show pipeline config
wiggum inspect agents                       # List agents
wiggum inspect activity [TASK-ID]           # View activity logs
WIGGUM_LOG_LEVEL=debug wiggum run           # Verbose logging
```

## Architecture

### Directory Structure

| Directory | Purpose |
|-----------|---------|
| `bin/` | CLI entry points (15 command scripts) |
| `lib/core/` | Shared utilities (logger, exit codes, file locking, preflight) |
| `lib/runtime/` | Backend-agnostic runtime abstraction (execution, retry, loop) |
| `lib/backend/` | Backend implementations (claude, opencode) |
| `lib/agents/` | Agent implementations (system, engineering, product) |
| `lib/pipeline/` | Pipeline engine (loader, runner - state machine) |
| `lib/worker/` | Worker lifecycle management |
| `lib/scheduler/` | Task scheduling, worker pool, priority, orchestrator functions |
| `lib/orchestrator/` | Orchestrator lifecycle, arg parsing, directory migration |
| `lib/tasks/` | Kanban parsing, task parsing |
| `lib/git/` | Git worktree operations, PR creation |
| `lib/utils/` | Logging, metrics, cost calculation |
| `config/` | Pipeline, agents, and schema definitions (JSON) |
| `spec/` | Architecture, pipeline schema, agent dev guide, protocol |
| `tests/` | Test suite (35+ test files) |
| `skills/` | Claude Code skill definitions |
| `tui/` | Python Textual-based monitoring UI |

### Runtime Abstraction (`lib/runtime/`)

Backend-agnostic execution layer. See `spec/RUNTIME-SCHEMA.md` for the full specification.

1. **runtime.sh** - Backend loader + public API (`run_agent_once`, `run_agent_resume`)
2. **runtime-loop.sh** - Iterative loop with optional supervision (`run_ralph_loop`)
   - `supervisor_interval=0` (default): Pure iterative loop with summaries
   - `supervisor_interval=N`: Supervisor reviews every N iterations (CONTINUE/STOP/RESTART)
3. **runtime-retry.sh** - Exponential backoff retry (`runtime_exec_with_retry`)
4. **backend-interface.sh** - Contract for backend implementations

**Backends** (`lib/backend/`):
- `claude/claude-backend.sh` - Claude Code CLI backend (default)
- `opencode/opencode-backend.sh` - OpenCode skeleton (stub)

**Backend selection** (priority): `WIGGUM_RUNTIME_BACKEND` env → `.ralph/config.json` → `config/config.json` → `"claude"`

### Pipeline Engine

Jump-based state machine defined in `config/pipelines/default.json`. Steps have `on_result` handlers (PASS/FAIL/FIX/SKIP) controlling flow via jump targets: `self`, `prev`, `next`, `abort`, or step ID.

Default pipeline: planning → execution (supervised) → summary → audit → audit-fix → test → docs → validation

### Agent Architecture

All agents source `lib/core/agent-base.sh` and implement:
```bash
agent_run(worker_dir, project_dir)
# Returns: 0=PASS, 10=FAIL, 12=MAX_ITERATIONS
```

**Agent Categories**:
- `lib/agents/system/` - task-worker, task-summarizer, resume-decide
- `lib/agents/engineering/` - software-engineer, security-audit, security-fix, test-coverage, validation-review
- `lib/agents/product/` - plan-mode, documentation-writer

### Worker Structure

Each worker operates in `.ralph/workers/worker-TASK-XXX-<timestamp>/` with:
- `workspace/` - Git worktree
- `prd.md` - Generated requirements
- `activity.jsonl` - Event log (NDJSON)
- `output/<agent>/` - Per-agent outputs
- `gate_result` - Last agent result (PASS/FAIL/FIX/SKIP)

### Worker Lifecycle State Machine (`config/worker-lifecycle.json`)

Workers transition through states via an event-driven state machine defined in `config/worker-lifecycle.json`. The spec is the single source of truth — adding states or transitions means editing JSON, not bash.

**Engine**: `lib/core/lifecycle-engine.sh` provides `emit_event(worker_dir, event, source, data_json)` — the only function needed to drive transitions. It is entirely generic: zero knowledge of specific states or events.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sig-id/chief-wiggum](https://github.com/sig-id/chief-wiggum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
