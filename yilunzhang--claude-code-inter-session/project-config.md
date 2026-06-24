---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

An agent-to-agent messaging bus for Claude Code: multiple CC sessions on
the same Unix machine connect to a localhost WebSocket server and
exchange messages that drive actions in the receiving session.

Two install modes, **both supported and tested**:

- **Plugin** (recommended): `/plugin marketplace add …` → `/plugin
  install inter-session@inter-session`, or `claude --plugin-dir <repo>`
  for local dev. Adds `userConfig` (port, idle-shutdown) and
  `monitors.json`. User invokes as `/inter-session:inter-session …`.
- **Standalone skill**: clone or symlink `skills/inter-session/` to
  `~/.claude/skills/inter-session/` (e.g. `ln -s
  <repo>/skills/inter-session ~/.claude/skills/inter-session`). The
  skill is self-contained — `bin/`, `requirements.txt`, and `SKILL.md`
  all live inside `skills/inter-session/`, so a copy or symlink of just
  that subdirectory is a fully working skill. User invokes as
  `/inter-session …` (no plugin namespace). No `userConfig`; override
  defaults via `INTER_SESSION_PORT` / `INTER_SESSION_IDLE_MINUTES` env
  vars if needed.

The skill content (`skills/inter-session/SKILL.md`) is install-mode
agnostic: the connect step has **no upfront dedup check**. It picks a
name and calls `Monitor()` directly. If a monitor is already running
for this CC session, `bin/client.py`'s ppid-flock catches the duplicate
and the new spawn exits with `[inter-session] another monitor for this
session is already running`, which the LLM surfaces via the Error
notifications path. Skipping the pre-check optimizes the common case
(not connected yet → straight spawn, ~50-100ms faster) and lets the
flock be the single source of truth for race-safety. Don't add a
`list.py --self` or `TaskList()` pre-check back into the connect step
— that was tried and reverted because the optimization paid more in
the common case than it saved in the edge case.

Layout follows the conventional `skills/<name>/SKILL.md` auto-discovery
pattern that the current CC plugin schema requires (`"skills": ["./"]`
is rejected with "Path escapes plugin directory"). **`bin/` lives
inside the skill dir** (`skills/inter-session/bin/`). The plugin's
monitor `when` defaults to `on-skill-invoke:inter-session` (lazy);
`bin/auto_start.py` flips it to `always` when the user runs
`/inter-session auto-start on`. Empirically `on-skill-invoke` may not
reliably auto-spawn a working monitor in current CC versions, so the
LLM's `Monitor()` call in the skill is what actually establishes the
connection most of the time.

When CLAUDE.md and other docs reference `bin/<script>.py` as an
abbreviated label, the actual path is
`skills/inter-session/bin/<script>.py`.

Single user, single machine. Unix-only (macOS / Linux / WSL2).

## Common commands

Local dev runs entirely in a project-local venv at `.venv`. The
Makefile bootstraps it on first use (uv preferred, stdlib `venv` as
fallback). System Python is never touched.

```bash
make test                                    # full suite (~49 s, 197 tests)
make test-fast                               # skip subprocess-spawning tests
make clean                                   # remove .venv
```

To run pytest with non-make flags, use the venv's pytest directly:

```bash
.venv/bin/pytest tests/test_server.py -v     # one file
.venv/bin/pytest -k "election" -v            # by substring
```

No build step, no linter configured. Runtime deps live at
`skills/inter-session/requirements.txt` (websockets + psutil); dev
deps inherit those plus pytest via `requirements-dev.txt`. Both reqs
files install into `.venv` via `make test` — there's nothing to install
by hand.

## Architecture (big picture)

Three process classes share a localhost WebSocket bus:

1. **`bin/server.py`** — single detached asyncio websockets server per
   port. Started by whichever client wins the `bind()` election. Owns
   the registry of connected agents, mints `msg_id`s, writes
   `messages.log`. Idle-shutdown after N minutes.

2. **`bin/client.py`** — long-lived per-session monitor. Each stdout
   line becomes a Claude Code notification. Manages reconnect with
   exponential backoff, ping/pong, and a ppid-based dedup flock.

3. **`bin/{send,list}.py`** — short-lived control CLIs. Connect with
   `role=control`, do not register as agents, never appear in `list`.
   Discover their owning session via `bin/discover.py` (process-tree
   walk + per-listener state file).

`bin/spawn.py` is the election + spawn boundary; `bin/shared.py` is
paths, validation, sanitizer, atomic bearer-token, identity helpers.

## Non-obvious invariants (read before changing the affected code)

### Race-free server election (`bin/spawn.py` + `bin/server.py --fd`)

The election is `bind()`-atomic: whoever wins spawns the server via
`subprocess.Popen(pass_fds=(fd,), start_new_session=True)`, and the
child adopts the bound fd with `socket.socket(fileno=fd).listen()`.
**PEP 446 is the gotcha**: CPython sets `FD_CLOEXEC` on sockets by
default, so `os.set_inheritable(fd, True)` is required — without it,
`execvp` silently closes the socket. `SO_REUSEADDR=1` is also set to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yilunzhang/claude-code-inter-session](https://github.com/yilunzhang/claude-code-inter-session) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
