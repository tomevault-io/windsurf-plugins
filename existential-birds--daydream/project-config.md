---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Daydream is an automated code review and fix loop using the Claude Agent SDK. It launches review agents equipped with Beagle skills (specialized knowledge modules) to review code, parse actionable feedback, apply fixes automatically, and validate changes by running tests.

## Commands

```bash
# Install dependencies and git hooks
make install
make hooks

# Run the CLI
daydream [TARGET] [OPTIONS]

# Run as module
python -m daydream

# Examples
daydream /path/to/project                          # Default: deep multi-stack loop
daydream --shallow -s python /path/to/project      # Shallow Python review-fix-test loop
daydream --review /path/to/project                 # Review only, skip fixes
daydream --comment --branch feat/x /path/to/project  # Post inline PR comments
daydream feedback 42 --bot "coderabbitai[bot]" /path/to/project  # Bot PR comments
daydream --trajectory /tmp/out.json /path/to/project  # Custom trajectory path

# Development
make lint       # Run ruff linter
make typecheck  # Run mypy type checker
make test       # Run pytest
make check      # Run all CI checks locally
```

## Architecture

The package follows a phased execution model:

```
cli.py → runner.py → phases.py → agent.py
                  ↘ ui.py (terminal output)
```

### Module Responsibilities

- **cli.py**: Entry point, argument parsing, signal handlers (SIGINT/SIGTERM)
- **runner.py**: Main orchestration via `run()` async function, `RunConfig` dataclass
- **phases.py**: Four workflow phases:
  1. `phase_review()` - Invoke Beagle review skill, write to `.review-output.md`
  2. `phase_parse_feedback()` - Extract actionable issues as JSON
  3. `phase_fix()` - Apply fixes one-by-one
  4. `phase_test_and_heal()` - Run tests, interactive retry/fix loop
- **agent.py**: Claude SDK client wrapper, `run_agent()` streams responses, `AgentState` dataclass for consolidated state, `MissingSkillError` exception
- **trajectory.py**: ATIF v1.6 trajectory recorder, `TrajectoryRecorder` with `ContextVar` propagation, `Invocation` per-`run_agent` scope, `Redactor` for secret scrubbing, `DaydreamPhase`/`DaydreamRunFlow` enums
- **ui.py**: Rich-based terminal UI with Dracula theme, live-updating panels
- **config.py**: Skill mappings, constants

### Key Patterns

- All agent interactions use `ClaudeSDKClient` from `claude-agent-sdk` with `bypassPermissions` mode
- Streaming responses are processed via async iterator over message types (AssistantMessage, UserMessage, ResultMessage)
- Tool call panels use Rich's `Live` for animated throbbers during execution
- Global state consolidated in `AgentState` dataclass (quiet_mode, model, shutdown_requested) with `_current_client` for SDK instance
- Trajectory recording via `TrajectoryRecorder` propagated through `ContextVar`; parallel fan-outs use `recorder.fork()` for sibling trajectory files

## Dependencies

- `claude-agent-sdk` - Claude Code SDK for agent interactions
- `anyio` - Async I/O abstraction (used for `anyio.run()`)
- `rich` - Terminal UI components
- `pyfiglet` - ASCII art header

## Prerequisites

Requires the Beagle plugin for Claude Code to be installed. The review skills (`beagle-python:review-python`, `beagle-react:review-frontend`, `beagle-elixir:review-elixir`) are provided by Beagle.

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Daydream**

Daydream is a Python CLI that automates code review and fix loops using the Claude Agent SDK. It launches review agents equipped with Beagle skills to review code, parse actionable feedback, apply fixes automatically, and validate by running tests. The default flow is a deep multi-stack pipeline; `--shallow` opts into a single-skill loop; `--comment` and `--review` turn the run into a review-only flow that posts to a PR or writes a report. The `daydream feedback <pr#>` subcommand ingests bot review comments. Each run is recorded as an **Agent Trajectory Interchange Format (ATIF v1.6)** trajectory, interoperable with the Harbor ecosystem.

**Core Value:** Reviews and recommendations must be grounded in actual codebase understanding — not guesses based on the diff alone. (Unchanged.) For this milestone specifically: **every daydream run produces a valid, replayable ATIF v1.6 trajectory that captures the full agent interaction history, tool I/O, and token/cost metrics.**

### Constraints

- **SDK**: Must continue using `claude-agent-sdk` for agent capabilities — no custom orchestration framework. ATIF is layered on top of the existing `Backend` / `AgentEvent` abstraction.
- **Backends**: Both Claude and Codex backends must produce ATIF trajectories. Codex parity is partial by design (no `cost_usd`, no `cached_tokens` from `turn.completed.usage`) — ATIF Metrics fields are all optional, so this is acceptable.
- **Existing tests**: All 343 tests must pass post-migration. New tests added for trajectory recording, redaction, and Harbor-golden round-trip validation.
- **CLI**: `--debug` is removed; `--trajectory <path>` is added. All output modes (`--comment`, `--review`, default loop) and the `daydream feedback` subcommand produce trajectories.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [existential-birds/daydream](https://github.com/existential-birds/daydream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
