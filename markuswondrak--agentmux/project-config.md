---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# Coding Agent Instructions

This file provides guidance to coding agents when working with code in this repository.

## Development

### Test-Driven Development

For every functional change, new feature, or bugfix, you MUST write or update tests FIRST — before implementing the change. This is a mandatory requirement:

1. **Write the test first** — Create or update tests that describe the expected behavior
2. **Run tests to confirm they fail** — Verify the test captures the issue or new requirement
3. **Implement the change** — Write the minimum code to make the test pass
4. **Run tests to confirm they pass** — Ensure all tests pass, including existing ones
5. **Refactor if needed** — Clean up code while keeping tests green

Do not skip this process. Tests are the specification of desired behavior.

### Install for development

```bash
python3 -m pip install -e ".[dev]"
```

### Run tests

```bash
pytest tests
```

### Lint and format

```bash
ruff check src tests           # lint
ruff check --fix src tests     # auto-fix
ruff format --check src tests  # check formatting
ruff format src tests          # format in place
```

### Pre-commit hooks

```bash
pre-commit run --all-files     # run all hooks
```

Hooks run automatically on commit: `ruff-check --fix` and `ruff-format`.
The `pip-compile` hooks regenerate `requirements.txt` and `requirements-dev.txt` when `pyproject.toml` changes.

### Config resolution

Default config resolution is layered:
- built-in defaults from `src/agentmux/configuration/defaults/config.yaml`
- optional user config from `~/.config/agentmux/config.yaml`
- project config from `.agentmux/config.yaml`
- explicit `--config <path>` override

## Architecture

This is a **tmux-based multi-agent orchestration system**. Instead of calling AI APIs directly, it drives existing CLI tools (`claude`, `codex`, `gemini`, `opencode`, `qwen`) by injecting keystrokes into tmux panes. This reuses existing OAuth-authenticated subscriptions rather than pay-per-token API calls.

### How it works

The pipeline application:
1. Creates a feature directory under `.agentmux/.sessions/<feature-name>/`
2. Spawns a tmux session with a **monitor pane** (left, 40 cols) and agent panes (right)
   - Pane border titles show the role name for each pane
3. Starts three event sources on a shared session event bus:
   - `FileEventSource` — watches the feature directory with `watchdog`, normalizes paths
   - `ToolCallEventSource` — tails `tool_events.jsonl` for MCP tool-call signals
   - `InterruptionEventSource` — polls for missing registered agent panes
4. `WorkflowEventRouter` routes events to phase-specific handler classes (e.g. `ArchitectingHandler`, `ReviewingHandler`), which emit structured workflow events that drive state transitions
5. Handlers build prompt files lazily just before dispatch, then send a file reference message (`Read and follow the instructions in /path/to/prompt.md`) to the appropriate tmux pane
6. The orchestrator persists state in `state.json` and advances the workflow based on handler-emitted events — not artifact detection

### State machine

The workflow progresses through these states (stored in `.agentmux/.sessions/<feature>/state.json`):

```
product_management? → architecting → planning → designing? → implementing → reviewing
    → verdict:pass → completing
    → verdict:fail → fixing → reviewing (review loop)
    → loop cap reached → completing
    → approval_received (done) OR changes_requested → architecting
```

Role routing in these phases:
- `product-manager`: product management phase only
- `architect`: architecting phase only — creates technical architecture document (the "What" and "With what")
- `planner`: planning/replanning only — creates execution plans from architecture (the "How" and "When")
- `designer`: designing phase only — creates `05_design/design.md` from plan with `needs_design: true`
- `reviewer`: reviewing (dynamically routed to specialized reviewers based on `execution_plan.yaml` `review_strategy`) and completing phases:
  - `reviewer_logic`: Logic & Alignment reviewer (functional correctness vs plan)
  - `reviewer_quality`: Quality & Style reviewer (clean code, naming, standards)
  - `reviewer_expert`: Deep-Dive Expert reviewer (security, performance, edge cases)
- `coder`: implementing/fixing

`state.json` persists the durable `phase` and optional metadata such as `last_event`, `review_iteration`, `subplan_count`, `product_manager`, `research_tasks` (a dict tracking code-researcher task status by topic), `web_research_tasks` (a dict tracking web-researcher task status by topic), and GitHub integration keys like `gh_available` / `issue_number`. Agents no longer write workflow statuses directly.

### Entry points

```
src/agentmux/pipeline/application.py      — CLI entry, launcher, --orchestrate mode
src/agentmux/configuration/__init__.py    — layered config, provider/model resolution
src/agentmux/shared/phase_catalog.py      — phase catalog: directories, flags, monitor ordering
src/agentmux/sessions/state_store.py      — session creation/resume, state.json lifecycle
src/agentmux/runtime/tmux_control.py      — tmux pane/session lifecycle, prompt dispatch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markuswondrak/AgentMux](https://github.com/markuswondrak/AgentMux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
