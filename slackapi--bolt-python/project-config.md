---
trigger: always_on
description: Slack Bolt for Python -- a framework for building Slack apps in Python.
---

# AGENTS.md - bolt-python

## Project Overview

Slack Bolt for Python -- a framework for building Slack apps in Python.

- **Foundation:** Built on top of `slack_sdk` (see `pyproject.toml` constraints).
- **Execution Models:** Supports both synchronous (`App`) and asynchronous (`AsyncApp` using `asyncio`) execution. Async mode requires `aiohttp` as an additional dependency.
- **Framework Adapters:** Features built-in adapters for web frameworks (Flask, FastAPI, Django, Tornado, Pyramid, and many more) and serverless environments (AWS Lambda, Google Cloud Functions).
- **Python Version:** Requires Python 3.7+ as defined in `pyproject.toml`.

- **Repository**: <https://github.com/slackapi/bolt-python>
- **Documentation**: <https://docs.slack.dev/tools/bolt-python/>
- **PyPI**: <https://pypi.org/project/slack-bolt/>
- **Current version**: defined in `slack_bolt/version.py` (referenced by `pyproject.toml` via `[tool.setuptools.dynamic]`)

## Environment Setup

You can verify the venv is active by checking `echo $VIRTUAL_ENV`. If tools like `black`, `flake8`, `mypy` or `pytest` are not found, ask the user to activate the venv.

A python virtual environment (`venv`) should be activated before running any commands.

```bash
# Create a venv (first time only)
python -m venv .venv

# Activate
source .venv/bin/activate

# Install all dependencies
./scripts/install.sh
```

## Common Commands

### Pre-submission Checklist

Before considering any work complete, you MUST run these commands in order and confirm they all pass:

```bash
./scripts/format.sh --no-install     # 1. Format
./scripts/lint.sh --no-install        # 2. Lint
./scripts/run_tests.sh <relevant>     # 3. Run relevant tests (see Testing below)
./scripts/run_mypy.sh --no-install    # 4. Type check
```

To run everything at once (installs deps + formats + lints + tests + typechecks):

```bash
./scripts/install_all_and_run_tests.sh
```

### Testing

Always use the project scripts instead of calling `pytest` directly:

```bash
# Run a single test file
./scripts/run_tests.sh tests/scenario_tests/test_app.py

# Run a single test function
./scripts/run_tests.sh tests/scenario_tests/test_app.py::TestApp::test_name
```

### Formatting, Linting, Type Checking

```bash
# Format -- Black, configured in pyproject.toml
./scripts/format.sh --no-install

# Lint -- Flake8, configured in .flake8
./scripts/lint.sh --no-install

# Type check -- mypy, configured in pyproject.toml
./scripts/run_mypy.sh --no-install
```

## Critical Conventions

### Sync/Async Mirroring Rule

**When modifying any sync module, you MUST also update the corresponding async module (and vice versa).** This is the most important convention in this codebase.

Almost every module has both a sync and async variant. Async files use the `async_` prefix alongside their sync counterpart:

```text
slack_bolt/middleware/custom_middleware.py          # sync
slack_bolt/middleware/async_custom_middleware.py    # async

slack_bolt/context/say/say.py                      # sync
slack_bolt/context/say/async_say.py                # async

slack_bolt/listener/custom_listener.py              # sync
slack_bolt/listener/async_listener.py              # async
```

**Modules that come in sync/async pairs:**

- `slack_bolt/app/` -- `app.py` / `async_app.py`
- `slack_bolt/middleware/` -- every middleware has an `async_` counterpart
- `slack_bolt/listener/` -- `listener.py` / `async_listener.py`, plus error/completion/start handlers
- `slack_bolt/listener_matcher/` -- `builtins.py` / `async_builtins.py`
- `slack_bolt/context/` -- each subdirectory (e.g., `say/`, `ack/`, `respond/`) has `async_` variants
- `slack_bolt/kwargs_injection/` -- `args.py` / `async_args.py`, `utils.py` / `async_utils.py`

**Adapters are an exception:** Most adapters are sync-only or async-only depending on the framework. Async-native frameworks (FastAPI, Starlette, Sanic, Tornado, ASGI, Socket Mode) have `async_handler.py`. Sync-only frameworks (Flask, Django, Bottle, CherryPy, Falcon, Pyramid, AWS Lambda, Google Cloud Functions, WSGI) have `handler.py`.

### Prefer the Middleware Pattern

Middleware is the project's preferred approach for cross-cutting concerns. Before adding logic to individual listeners or utility functions, consider whether it belongs as a built-in middleware in the framework.

**When to add built-in middleware:**

- Cross-cutting concerns that apply to many or all requests (logging, metrics, observability)
- Request validation, transformation, or enrichment
- Authorization extensions beyond the built-in `SingleTeamAuthorization`/`MultiTeamsAuthorization`
- Feature-level request handling (the `Assistant` middleware in `slack_bolt/middleware/assistant/assistant.py` is the canonical example -- it intercepts assistant thread events and dispatches them to registered sub-listeners)

**How to add built-in middleware:**

1. Subclass `Middleware` (sync) and implement `process(self, *, req, resp, next)`. Call `next()` to continue the chain.
2. Subclass `AsyncMiddleware` (async) and implement `async_process(self, *, req, resp, next)`. Call `await next()` to continue.
3. Export from `slack_bolt/middleware/__init__.py` (sync) and `slack_bolt/middleware/async_builtins.py` (async).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slackapi/bolt-python](https://github.com/slackapi/bolt-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
