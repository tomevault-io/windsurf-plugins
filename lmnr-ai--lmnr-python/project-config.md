---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Laminar Python SDK - tracing and evaluation framework for LLM applications. Provides automatic instrumentation for LLM providers (OpenAI, Anthropic, Groq, etc.), manual tracing via decorators, and an evaluation framework for testing LLM outputs.

## Commands

```bash
# Install dependencies with all extras
uv sync --all-extras --dev

# Run all tests
uv run pytest

# Run specific test file
uv run pytest tests/test_initialize.py

# Run specific test
uv run pytest tests/test_initialize.py::test_laminar_initialize_url_parsing -v

# Lint
uv run flake8 src/

# Format
uv run autopep8 --in-place --aggressive src/lmnr/**/*.py

# CLI commands
lmnr eval <file.py>           # Run evaluations
lmnr dev <file.py> --function <name>  # Interactive debugger
lmnr datasets pull <id>       # Pull dataset
```

## Architecture

### Core SDK (`src/lmnr/sdk/`)
- `laminar.py` - Main `Laminar` class with `initialize()` entry point and span management
- `decorators.py` - `@observe()` decorator for manual instrumentation
- `evaluations.py` - `evaluate()` function and `Evaluation` class for running tests
- `client/` - HTTP clients (`LaminarClient`, `AsyncLaminarClient`) for API interactions

### OpenTelemetry Layer (`src/lmnr/opentelemetry_lib/`)
- `tracing/instruments.py` - `Instruments` enum listing all auto-instrumentable providers
- `tracing/__init__.py` - `TracerWrapper` singleton managing OTEL setup
- `opentelemetry/instrumentation/` - Provider-specific instrumentors (anthropic/, openai/, groq/, etc.)
- Each provider instrumentor has: `__init__.py` (instrumentor class), `utils.py` (parsing), `span_utils.py` (span creation)

### CLI (`src/lmnr/cli/`)
- `evals.py` - `lmnr eval` command
- `dev.py` - `lmnr dev` interactive debugger

## Key Patterns

- **Singleton**: `Laminar` and `TracerWrapper` are singletons initialized once at startup
- **Provider instrumentors** extend OpenTelemetry's base instrumentor and are dynamically initialized based on `Instruments` enum
- **Tests use VCR.py** to record/replay HTTP responses in `tests/cassettes/`
- **Tests use InMemorySpanExporter** configured in `tests/conftest.py` for span assertions

## Dev dependencies

- All entries in the `[dependency-groups].dev` section of `pyproject.toml` MUST be pinned to a specific version with `==X.Y.Z`. Do NOT use unbounded specifiers (`>=`, `^`, `~`, `<`, ranges, or bare package names). Pinning keeps the test matrix deterministic across developers and CI. When adding a new dev dep, look up the current release on https://pypi.org and pin to that exact version; bumps then go through a normal PR.

## Environment Variables

```
LMNR_PROJECT_API_KEY  # API key (can also pass to initialize())
LMNR_BASE_URL         # API base URL (default: https://api.lmnr.ai)
```

## Instrumentation tests (VCR)

- Tests under `tests/test_instrumentations/**` replay via VCR cassettes in `cassettes/<module-name>/`. Sensitive headers are filtered by the root `vcr_config` (`authorization`, `api-key`, `x-api-key`, `x-goog-api-key`).
- The sandbox sets `ANTHROPIC_BASE_URL` / `ANTHROPIC_BEDROCK_BASE_URL` to a local proxy; when recording new anthropic cassettes, **unset those two env vars** first or the cassette will record `http://127.0.0.1:...` and fail to replay against `api.anthropic.com`. Use `unset ANTHROPIC_BASE_URL ANTHROPIC_BEDROCK_BASE_URL` before running `pytest --record-mode=once`.
- Re-record a single module with `rm -rf tests/.../cassettes/<module>/ && pytest <path> --record-mode=once`. VCR's default `record_mode` is `once`, so replays work without flags as long as the cassette exists.

## Anthropic instrumentor

- `messages.parse` (the structured-output helper introduced in `anthropic>=0.59`) is wrapped with the same `_wrap` / `_awrap` that wrap `messages.create` — the resulting span name is `anthropic.chat` and the attribute set is identical, plus `gen_ai.request.structured_output_schema`.
- Schema extraction lives in `span_utils._extract_structured_output_schema`. It handles: (1) `output_format=PydanticModel` via `model_json_schema()`, (2) `output_format=<any TypeAdapter-compatible type>` via `pydantic.TypeAdapter(...).json_schema()`, (3) raw-dict `output_format` (bare schema or `JSONOutputFormatParam`), and (4) `output_config={"format": {"type": "json_schema", "schema": {...}}}` on `messages.create`.
- `anthropic.lib.bedrock._beta_messages.Messages` does **not** expose `parse` (only regular + `beta.messages.messages` do). `_instrument` already swallows `ModuleNotFoundError` / `AttributeError` so the missing bedrock.parse attempt is a silent no-op.

## deepagents instrument

- `Instruments.DEEPAGENTS` is auto-enabled when `deepagents` is installed. When auto-enabled, `LANGCHAIN` and `LANGGRAPH` are auto-removed from the default instrument set (see `_DEEPAGENTS_NOISE_CONFLICTS` in `tracing/instruments.py`) — the LangSmith-style node-level spans they emit add no signal on top of what `LaminarMiddleware` already captures, and clutter the transcript view.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lmnr-ai/lmnr-python](https://github.com/lmnr-ai/lmnr-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
