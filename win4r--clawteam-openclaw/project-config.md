---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

`clawteam` is a Python CLI for orchestrating swarms of CLI coding agents (OpenClaw, Claude Code, Codex, Hermes, nanobot, Cursor, etc.). This repo is `win4r/ClawTeam-OpenClaw` — a fork of `HKUDS/ClawTeam` where the **default agent is `openclaw`** instead of `claude`. The `+openclaw` PEP 440 local identifier in the version (e.g. `0.3.0+openclaw1`) must be preserved on bumps.

There is no server and no database. All state is JSON files under `~/.clawteam/` (overridable via `CLAWTEAM_DATA_DIR`). Agents coordinate via the shared filesystem: kanban tasks, per-recipient inboxes, team configs, git worktree registries.

## Common commands

```bash
# Dev install (from repo root)
pip install -e ".[dev]"          # dev deps: pytest, ruff
pip install -e ".[p2p]"          # optional ZeroMQ P2P transport

# Lint + test — same commands CI runs (.github/workflows/ci.yml)
ruff check clawteam/ tests/
pytest tests/                    # full suite
pytest tests/test_spawn_backends.py -v        # one file
pytest tests/test_board.py::test_foo -v        # one test
pytest -k "mailbox"              # by name

# CLI entry points
clawteam <subcommand>            # installed script
python -m clawteam <subcommand>  # module entry (clawteam/__main__.py)
```

Tests auto-isolate: `tests/conftest.py` monkeypatches both `CLAWTEAM_DATA_DIR` and `HOME` to `tmp_path`, so no test can touch real `~/.clawteam/` or `~/.clawteam/config.json`.

## Architecture — the big picture

### CLI surface

**All commands live in a single file: `clawteam/cli/commands.py` (~2700 lines).** It uses typer sub-apps:

| Sub-app | Purpose |
|---------|---------|
| `config` | `show` / `set` / `get` / `health` |
| `team` | `spawn-team` / `discover` / `status` / `request-join` / `approve-join` / `cleanup` |
| `inbox` | `send` / `broadcast` / `receive` / `peek` / `log` / `watch` |
| `task` | `create` / `get` / `update` / `list` / `stats` / `wait` |
| `cost` | `report` / `show` / `budget` |
| `session` | `save` / `show` / `clear` |
| `plan` | `submit` / `approve` / `reject` |
| `lifecycle` | `request-shutdown` / `approve-shutdown` / `idle` / `on-exit` / `check-zombies` |
| `runtime` | tmux-only live injection: `inject` / `watch` / `state` |
| `identity` | `show` / `set` |
| `board` | `show` / `overview` / `live` / `serve` (web UI) / `attach` (tmux tiled view) |
| `workspace` | `list` / `checkpoint` / `merge` / `cleanup` / `status` |
| `template` | `list` / `show` |
| (top-level) | `spawn` and `launch` |

Subtle trap: `spawn` takes `--team` / `-t`, but `launch` takes `--team-name` / `-t`. These flag names are intentionally different and appear in user docs — do not "harmonize" them.

The `_output(data, human_fn)` helper picks JSON vs. human-readable output based on the global `--json` flag. Always call it instead of `print()` for command results.

### Two pluggable backends

**Transport** (`clawteam/transport/`) — how messages move between agents.
- `Transport` ABC → `FileTransport` (default, writes `inboxes/{agent}/msg-*.json` atomically) or `P2PTransport` (ZMQ PUSH/PULL with FileTransport fallback, requires `[p2p]` extra).
- `MailboxManager` in `clawteam/team/mailbox.py` builds `TeamMessage` pydantic models, serialises, delegates bytes to the transport.
- Selection: `CLAWTEAM_TRANSPORT` env var → config → `"file"`.

**TaskStore** (`clawteam/store/`) — kanban task persistence.
- `BaseTaskStore` ABC → `FileTaskStore` (only implementation today). Shim at `clawteam/team/tasks.py` preserves the legacy `TaskStore` import path.
- Selection: `CLAWTEAM_TASK_STORE` env var → config → `"file"`.

Both factories live in `clawteam/{transport,store}/__init__.py`.

### Spawn system

Splits cleanly in two:

1. **Backend** (`clawteam/spawn/{base,tmux_backend,subprocess_backend}.py`) — how the process is launched.
   - `SpawnBackend` ABC → `TmuxBackend` (one tmux session per team, `clawteam-{team}`, one window per agent) or `SubprocessBackend`.
   - Factory: `get_backend(name)` in `clawteam/spawn/__init__.py`. `normalize_backend_name()` forces `tmux → subprocess` on Windows.
   - Default per-platform from `platform_compat.default_spawn_backend()`: `tmux` on Unix, `subprocess` on Windows.
   - `spawn_with_retry()` wraps a backend with exponential-backoff retry.

2. **Command prep** (`clawteam/spawn/adapters.py`) — how CLI flags/args are assembled.
   - `NativeCliAdapter.prepare_command()` is the dispatcher. Each supported CLI has an `is_foo_command()` predicate in `adapters.py` + `command_validation.py`, and its own branch for flag injection:
     - `claude`/`qwen` → append `--dangerously-skip-permissions` when `skip_permissions`.
     - `codex` → `--dangerously-bypass-approvals-and-sandbox`.
     - `gemini`/`kimi`/`opencode`/`hermes` → `--yolo`.
     - `hermes` → insert `chat` subcommand if command is bare `hermes`, add `--source tool`, pass prompt via `-q` (NOT `--continue`, which resumes an existing session).
     - `openclaw` → `--session <name>`, `--message <prompt>` (or `--local --session-id` when using the `agent` subcommand).
     - `kimi` / `nanobot` → `-w <cwd>` + `--print -p` / `-m`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [win4r/ClawTeam-OpenClaw](https://github.com/win4r/ClawTeam-OpenClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
