---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development practices

These rules are non-negotiable for all work in this repository.

**No hardcoded environment values.** Every configurable value — URLs, model names, timeouts, feature flags — must be read from environment variables or a `FailureDiagnosticsConfig`-style config object. Never embed literals that differ between environments directly in code.

**All functions async by default.** Write `async def` unless the function genuinely cannot be async (e.g. a `__exit__` protocol method, a `ContextVar` accessor, or a pure data transformation with no I/O). When wrapping a sync-only third-party call, use `asyncio.to_thread`. Document the reason in a comment when sync is unavoidable.

**Distribute work across subagents and run in parallel.** When implementing a task that has independent parts — separate files, separate test suites, separate concerns — spawn subagents and run them concurrently rather than sequentially. Never do serially what can be done in parallel.

**Git: never push directly to `main`.** Always work on `dev` (create it if it doesn't exist). Push to `dev`, then open a PR to `main`. The PR requires human approval before merging. This applies to all commits — features, fixes, docs, and chores alike.

**Do not reference this file (or any internal guidance file) in code, docs, or wikis.** CLAUDE.md is an internal tool for AI assistance only. It must not appear in source code comments, README, wiki pages, or any user-facing content.

---

## Commands

```bash
# Install dependencies (includes typer, uvicorn, fastapi, pydantic for examples)
uv sync --group dev

# Copy env var template (all vars are optional)
cp .env.example .env

# Run examples
uv run python examples/basic.py
uv run python examples/success.py
uv run python examples/basic_ollama.py  # requires RUNTIME_NARRATIVE_MODEL

# Auto-instrumentation examples (Phase 1)
uv run python examples/narrative_class.py
uv run python examples/instrument_module.py
uv run python examples/auto_instrument.py

# Run FastAPI demo
uv run python -m examples.fastapi_app.run
# With Ollama failure analysis:
RUNTIME_NARRATIVE_MODEL=llama3 uv run python -m examples.fastapi_app.run
# With a custom endpoint (vLLM, llama.cpp, etc.):
RUNTIME_NARRATIVE_MODEL=llama3 RUNTIME_NARRATIVE_ENDPOINT=http://localhost:8000/api/generate uv run python -m examples.fastapi_app.run

# Rich failure diagnostics (locals, etc.) via env
RUNTIME_NARRATIVE_FAILURE_DIAGNOSTICS=rich uv run python examples/basic.py

# Run all tests
uv run pytest tests/ -v

# Run a single test file
uv run pytest tests/test_story.py -v

# Run a single test by name
uv run pytest tests/test_diagnostics.py -v -k "test_name"
```

## Environment variables

| Variable | Values | Default | Effect |
|---|---|---|---|
| `RUNTIME_NARRATIVE_ENV` | `development`, `production` | `development` | Production caps traceback to 8 000 chars and forces lean mode |
| `RUNTIME_NARRATIVE_FAILURE_DIAGNOSTICS` | `lean`, `rich` | `lean` | `rich` captures locals for up to 2 frames |
| `RUNTIME_NARRATIVE_ALLOW_RICH_IN_PRODUCTION` | `1`, `true` | off | Bypass production safeguard for rich diagnostics |
| `RUNTIME_NARRATIVE_MODEL` | model name string | — | Used by example scripts to pick an Ollama/LLM model |
| `RUNTIME_NARRATIVE_ENDPOINT` | URL | — | Custom LLM endpoint for example scripts |
| `RUNTIME_NARRATIVE_RICH_LOG_FILE` | file path | — | Read by `renderer_defaults.default_renderers()` (shared by all auto-instrumentation entry points). Adds a `ConsoleRenderer` writing to this file, on top of the TTY-based base selection |
| `RUNTIME_NARRATIVE_RICH_LOG_CONSOLE` | `1`, `0` | `1` | With `RUNTIME_NARRATIVE_RICH_LOG_FILE` set and stdout a TTY, `0` suppresses the console copy so the narrative goes to the file only |

## Architecture

The library (`runtime_narrative/`) models execution as **stories** composed of **stages**, emitting lifecycle events that renderers consume.

### Core execution flow

1. `story(name)` — dual sync/async context manager (`with` / `async with`) that creates a `StoryRuntime`, sets it on `current_story` (a `ContextVar`), and emits `StoryStarted`/`StoryCompleted` events. On exception, builds enriched failure data (see **Diagnostics** below), optionally runs failure analysis, and emits `FailureOccurred` before `StoryCompleted`. Accepts `module: str | None = None`; when omitted, the calling module is auto-detected via a single `sys._getframe(1)` lookup at construction time (cheap — no stack walking) and stored on `StoryStarted.module`. Wrappers that open `story()` on a caller's behalf (decorators, middleware) should pass `module=` explicitly — `@runtime_narrative_story` passes `func.__module__` for this reason.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sraj0501/runtime_narrative](https://github.com/sraj0501/runtime_narrative) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
