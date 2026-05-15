---
trigger: always_on
description: Orientation for AI agents working on this repo. Pair this with `README.md`
---

# AGENTS.md

Orientation for AI agents working on this repo. Pair this with `README.md`
(end-user docs) and the source itself — the code is the source of truth.

## What this is

`symphony-linear` is a single-process Python daemon that orchestrates AI work on
Linear or GitHub issues (one backend per daemon instance). The loop:

1. Poll the tracker for issues with the trigger signal (label or project
   field, depending on backend).
2. For each new issue: clone the project's repo into a per-ticket workspace,
   switch to the ticket branch, optionally run `.symphony/setup`, then run
   `opencode run` inside a bubblewrap sandbox with the ticket title +
   description as the prompt.
3. Post the AI's final message as a comment and transition the ticket
   to the configured "needs input" state.
4. When a human comments, resume the OpenCode session with the new comment as
   user input, post the result, repeat.

There is no web UI, no API, no database. State lives in `state.json` in the
workspace dir. The only external services are the issue tracker (Linear or
GitHub, accessed via GraphQL) and OpenCode (launched as a subprocess inside
`bwrap`).

## Stack

- Python 3.11+, packaged with `hatchling` (see `pyproject.toml`).
- `uv` for dependency management (`uv.lock` is committed). The venv is at
  `.venv/` — use `.venv/bin/python` and `.venv/bin/pytest` directly.
- Runtime deps: `pyyaml`, `pydantic` v2, `httpx`.
- Dev deps: `pytest`.
- External binaries required at runtime: `bwrap`, `git`, `opencode`.
- CLI entry point: `symphony` → `symphony_linear.cli:main`.
- Backends: the orchestrator talks to a `Tracker` protocol (see `tracker.py`).
  Concrete implementations exist for Linear (`linear_tracker.py`) and
  GitHub Projects v2 (`github_tracker.py`). Exactly one backend is active
  per daemon, selected at config time.

## Layout

```
symphony_linear/
  cli.py            argparse + wiring; loads config, builds Orchestrator, runs it
  config.py         YAML + pydantic config; ~ and ${VAR} expansion; token fallback
  state.py          TicketState model + StateManager (atomic JSON writes, threading.Lock)
  tracker.py        Tracker-neutral protocol, errors, and enums (the seam for orchestrator)
  linear.py         Linear GraphQL client (httpx, sync); typed exceptions; Issue/Comment/Project models
  linear_tracker.py Linear backend adapter implementing the Tracker protocol
  github.py         GitHub GraphQL client (httpx, sync); typed exceptions
  github_tracker.py GitHub Projects v2 backend adapter implementing the Tracker protocol
  sandbox.py        Single function: run_in_sandbox() → builds the bwrap argv and returns a Popen
  opencode.py       run_initial / run_resume; parses OpenCode's NDJSON event stream
  workspace.py      prepare() / remove(): clone, branch switch, .symphony/setup; path-containment check
  orchestrator.py   The brain: poll loop, per-ticket pipelines, ThreadPoolExecutor, cancellation
  logging.py        stderr logging setup
tests/              pytest, mostly unit with mocks; integration tests marked `integration` (shell out to `bwrap`/`git` — they never call the real `opencode` binary or any LLM)
```

The flow worth knowing: `orchestrator._tick()` is called every
`poll_interval_seconds`. It fans out work via `_schedule_task()` onto a
5-worker `ThreadPoolExecutor`, with per-ticket serialization (a ticket only
gets one task in flight at a time). Subprocesses are tracked in
`_subprocesses` so they can be killed when a ticket is cleaned up (daemon
shutting down, or the ticket is no longer triggered — see `_is_still_triggered`).

## Key invariants and gotchas

- **TicketStatus is daemon-internal**, distinct from Linear workflow states.
  Don't conflate `TicketStatus.needs_input` (in `state.json`) with the Linear
  state named "Needs Input". The same applies to GitHub: the daemon's internal
  status is separate from the project's Status field value.
- **The daemon polls tickets in `in_progress_state`, `needs_input_state`, and
  (if configured) `qa_state`** (see `_fetch_triggered_issues`). When a human
  comments on a `needs_input` ticket, `_resume_pipeline` transitions it back
  to `in_progress` itself — users don't need to do that manually. Human
  comments on a ticket in `qa_state` also trigger the normal resume pipeline:
  the ticket is transitioned to `in_progress_state`, the agent runs, the
  ticket lands in `needs_input_state`, and the existing `_reconcile_serve`
  logic kills the active serve on the next tick because its owner left QA.
- **The bot's own comments are filtered out** via the bot user id (`viewer.id`
  cached on the client). New "human" comments = comments whose
  `user_id != bot_user_id`. The `bot_user_email` in the config exists for
  documentation; the actual matching is by id. GitHub uses the viewer's node
  id — the same filtering logic applies.
- **State entry exists ⟺ workspace exists.** `orchestrator._tick` step 3
  fires cleanup (cancel subprocesses, remove state entry, remove workspace)
  whenever a tracked ticket is no longer *triggered* — i.e. the trigger label
  is absent, the Linear state is no longer an active state, the ticket is
  archived, or the ticket was deleted. A re-trigger of the same ticket after

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skorokithakis/symphony](https://github.com/skorokithakis/symphony) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
