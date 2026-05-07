---
trigger: always_on
description: Claude Code adaptation of [OpenAI's Symphony](https://github.com/openai/symphony). Orchestrates Claude Code agents via Linear issues.
---

# Stokowski

Claude Code adaptation of [OpenAI's Symphony](https://github.com/openai/symphony). Orchestrates Claude Code agents via Linear issues.

This file is the single source of truth for contributors. It covers architecture, design decisions, key behaviours, and how to work on the codebase.

---

## What it does

Stokowski is a long-running Python daemon that:
1. Polls Linear for issues in configured active states
2. Creates an isolated git-cloned workspace per issue
3. Launches Claude Code (`claude -p`) in that workspace
4. Manages multi-turn sessions via `--resume <session_id>`
5. Retries failures with exponential backoff
6. Reconciles running agents against Linear state changes
7. Exposes a live web dashboard and terminal UI

The agent prompt, runtime config, and workspace setup all live in `workflow.yaml` in the operator's directory — not in this codebase.

---

## Package structure

```
stokowski/
  config.py        workflow.yaml parser + typed config dataclasses
  linear.py        Linear GraphQL client (httpx async)
  models.py        Domain models: Issue, RunAttempt, RetryEntry
  orchestrator.py  Main poll loop, dispatch, reconciliation, retry
  prompt.py        Three-layer prompt assembly for state machine workflows
  runner.py        Claude Code CLI integration, stream-json parser
  tracking.py      State machine tracking via structured Linear comments
  workspace.py     Per-issue workspace lifecycle and hooks
  web.py           Optional FastAPI dashboard
  main.py          CLI entry point, keyboard handler
  __main__.py      Enables python -m stokowski
```

---

## Key design decisions

### Claude Code CLI instead of Codex app-server
Symphony uses Codex's JSON-RPC `app-server` protocol over stdio. Stokowski uses Claude Code's CLI:
- First turn: `claude -p "<prompt>" --output-format stream-json --verbose`
- Continuation: `claude -p "<prompt>" --resume <session_id> --output-format stream-json --verbose`

`--verbose` is required for `stream-json` to work. `session_id` is extracted from the `result` event in the NDJSON stream.

### Python + asyncio instead of Elixir/OTP
Simpler operational story — single process, no BEAM runtime, no distributed concerns. Concurrency via `asyncio.create_task`. Each agent turn is a subprocess launched with `asyncio.create_subprocess_exec`.

### No persistent database
All state lives in memory. The orchestrator recovers from restart by re-polling Linear and re-discovering active issues. Workspace directories on disk act as durable state.

### workflow.yaml as the operator contract
The operator's `workflow.yaml` defines the runtime config and state machine. Stokowski re-parses it on every poll tick — config changes take effect without restart. Both `.yaml` and legacy `.md` (YAML front matter + Jinja2 body) formats are supported. Prompt templates are now separate `.md` files referenced by path from the config.

### State machine workflow
Each workflow defines a set of internal states that map to Linear states. States have types: `agent` (runs Claude Code), `gate` (waits for human review), or `terminal` (issue complete). Transitions between states are declared explicitly in config.

**Three-layer prompt assembly:** Every agent turn's prompt is built from three layers concatenated together:
1. **Global prompt** — shared context loaded from a `.md` file (referenced by `prompts.global_prompt`)
2. **Stage prompt** — state-specific instructions loaded from the state's `prompt` path
3. **Lifecycle injection** — auto-generated section with issue metadata, transitions, rework context, and recent comments

**Gate protocol:** When an agent completes a state that transitions to a gate, Stokowski moves the issue to the gate's Linear state and posts a structured tracking comment. Humans approve or request rework via Linear state changes. On approval, Stokowski advances to the gate's `approve` transition target. On rework, it returns to the gate's `rework_to` state.

**Structured comment tracking:** State transitions and gate decisions are persisted as HTML comments on Linear issues (`<!-- stokowski:state {...} -->` and `<!-- stokowski:gate {...} -->`). These enable crash recovery and provide context for rework runs.

### Workspace isolation
Each issue gets its own directory under `workspace.root`. Agents run with `cwd` set to that directory. Workspaces persist across turns for the same session; they're deleted when the issue reaches a terminal state.

### Headless system prompt
Every first-turn launch appends a system prompt via `--append-system-prompt` that instructs Claude not to use interactive skills, slash commands, or plan mode. This prevents agents from stalling on interactive workflows.

---

## Component deep-dives

### config.py
Parses `workflow.yaml` (or legacy `.md` with front matter) into typed dataclasses:
- `TrackerConfig` — Linear endpoint, API key, project slug
- `PollingConfig` — interval
- `WorkspaceConfig` — root path (supports `~` and `$VAR` expansion)
- `HooksConfig` — shell scripts for lifecycle events + timeout (includes `on_stage_enter`)
- `ClaudeConfig` — command, permission mode, model, timeouts, system prompt
- `AgentConfig` — concurrency limits (global + per-state)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sugar-Coffee/stokowski](https://github.com/Sugar-Coffee/stokowski) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
