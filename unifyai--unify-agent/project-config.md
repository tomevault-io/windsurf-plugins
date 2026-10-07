---
trigger: always_on
description: GENERATED FILE - DO NOT EDIT DIRECTLY.
---

<!--
    GENERATED FILE - DO NOT EDIT DIRECTLY.

    Regenerate with:  python3 .agents/global-rules/build_agents_md.py

    Edit the sources instead:
      .agents/repo.md              this repo's overview and always-on guidance
      .agents/rules/*.md           this repo's own rules
      .agents/shared.txt           which shared rules this repo includes
      .agents/global-rules/rules/  rules shared across all unifyai repos
                                   (submodule: unifyai/global-agent-rules)
-->

# Unify

The brain of a local AI assistant: a persistent conversation loop above a
code-writing actor, two skill libraries, and one SQLite store.

Read [`ARCHITECTURE.md`](ARCHITECTURE.md) first for the system design. This file
covers *how to work on the code*, not *what the code does*.

## What Unify is

Unify implements an AI assistant's brain as a persistent **conversation loop** above a code-writing **`Actor`**, with two **skill libraries** behind it: `FunctionManager` (stored Python functions, the *what*) and `GuidanceManager` (procedures, the *how*). The actor discovers skills before it writes code, runs plans in a persistent Python sandbox, and a storage review after each run distils what worked back into the libraries. Manager methods run inside an **async LLM tool loop** and return a **steerable handle** that supports `ask`, `interject`, `pause`, `resume`, `stop` — all the way down the nesting tree. The skill libraries expose direct CRUD methods as Actor JSON tools (`FunctionManager_*`, `GuidanceManager_*`).

Everything persists in one SQLite file, `unify/db.py` (imported as `from unify import db`): five tables (`functions`, `primitives`, `guidance`, `builtin_guidance`, `messages`) and two views (`all_functions`, `all_guidance`) over each catalogue and its seeded rows. Managers write plain SQL; every filter the model writes is a SQL `WHERE` clause run read-only. There is no backend service, no accounts and no infrastructure: the only external dependency is an LLM provider, reached through the sibling `unillm` repo (editable install via `[tool.uv.sources]` in `pyproject.toml`) for chat, and directly through OpenRouter for the embeddings behind skill search (`unify/common/embeddings.py`; `UNIFY_LOCAL_EMBEDDINGS` computes them in process instead).

The assistant is **reactive**: it acts on the messages it receives and on the work those messages start. There is no scheduler, no timer wheel and no inbound channel other than the in-app chat.

## Run the agent locally

```bash
uv sync --all-groups
cp .env.example .env        # add one LLM provider key
.venv/bin/python -m unify   # chat; `unify` is the same entry point on PATH inside the venv
```

- The runtime's home is `UNIFY_HOME` (default `~/.unify`): `store.sqlite`, `embeddings.sqlite` (a cache of the vectors skill search compares, safe to delete), `workspace/` (the actor's working directory, attachments and downloads) and `logs/`.
- `unify --debug` streams runtime logs to the terminal as well as the log files; `unify --home DIR` uses another home.
- The chat CLI lives in `unify/cli.py`. It publishes `UnifyMessageReceived` events on the in-memory event broker and renders the `UnifyMessageSent` replies, so it is one front end over the same loop any client can drive.
- `unify act "request"` bypasses the conversation loop: one `CodeActActor` takes the request, progress streams to stderr from the handle's notification queue, a question the actor asks is answered from the terminal through `next_clarification`/`answer_clarification`, and the result goes to stdout (`--json` adds token accounting, `--persist` keeps the sandbox for follow-ups, `--no-store` skips the storage review). Use it for like-for-like comparisons with single-loop harnesses and for benchmark runners.

## Development environment

```bash
# First-time setup (fresh clone)
pip install uv && uv sync --all-groups
```

- **Python interpreter:** always use `.venv/bin/python`. Never the system Python.
- **Bootstrap:** if `.venv/` is missing, `uv sync --all-groups` recreates it.
- **`uv.lock` is protected** — never hand-edit it. Use the package manager.
- **Do not output `.env` or `*.key` contents to chat.**

### Running tests

Tests run in tmux sessions, each test in its own session against its own SQLite store, with logs streamed to `logs/pytest/`. The runner blocks until everything completes.

```bash
# Default — one session per test, max concurrency
tests/parallel_run.sh tests/function_manager/

# Specific test
tests/parallel_run.sh tests/function_manager/storage/test_primitives.py::test_name

# Serial mode (one session per file) for large suites
tests/parallel_run.sh -s tests/

# With timeout
tests/parallel_run.sh --timeout 300 tests/function_manager/
```

Each agent (or terminal) gets an **isolated tmux server automatically**, so concurrent agents don't collide.

### When a test fails

1. **Never inspect tmux panes directly.** Read the corresponding log in `logs/pytest/<YYYY-MM-DDTHH-MM-SS_socket>/`.
2. **Use `Read` (not `cat`/`tail`)** — `logs/` is gitignored, so `Grep`/`Glob` won't find files there.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unifyai/unify-agent](https://github.com/unifyai/unify-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
