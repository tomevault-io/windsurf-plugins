---
trigger: always_on
description: This guide explains how to work on the caddy-snake project: environment setup, testing, debugging, profiling, and benchmarks.
---

# AGENTS.md — Working on caddy-snake

This guide explains how to work on the caddy-snake project: environment setup, testing, debugging, profiling, and benchmarks.

---

## Pre-commit checklist

Before committing, run the full local QA suite (recommended):

```bash
./scripts/qa.sh
```

Or run checks individually:

1. **Pre-commit hooks**: `pre-commit run --all-files`
2. **Go tests**: `go test -race -v .`
3. **Go + caddytest** (in-process Caddy, on-demand TLS HTTPS): `go test -race -tags=caddytest -timeout 180s .`
4. **Python tests**: `pytest caddysnake_test.py -v`
5. **Static checks**: `golangci-lint run ./...`, `ruff check .`, `ty check` (or `uvx ty==0.0.55 check`)
6. **Integration tests** — at minimum **Flask** and **FastAPI**:
   - `./tests/integration.sh flask 3.13`
   - `./tests/integration.sh fastapi 3.13`
   - For **shared cache** changes: `./tests/integration.sh simple_cache 3.13`
7. **Embed-app** (optional, requires network): `cd cmd/embed-app && ./build.sh app.zip 3.13 && ./test_embed.sh embed-test`

Install hook once: `pre-commit install`

See [Running tests](#running-tests) and [Automated quality assurance](#automated-quality-assurance) for details.

---

## Documentation

When you implement or change a **user-facing feature** (Caddyfile directives, worker env vars, Python/Go APIs, cache protocol, CLI behavior, integration-test apps, etc.), **update the docs in the same PR** before merge:

1. **[`docs/docs/reference.md`](docs/docs/reference.md)** — authoritative configuration and API reference (Read the Docs).
2. **[`README.md`](README.md)** — overview and quick examples for GitHub visitors.
3. **[`AGENTS.md`](AGENTS.md)** — only when agent/workflow guidance changes (setup, QA, release steps).

For cache or multi-worker features, also cover: env vars, wire protocol (`CS*` commands if applicable), Python API, limits, security/trust model, and an integration test under `tests/` when behavior spans workers.

Build docs locally if you touch MkDocs content: `cd docs && mkdocs build` (also exercised in CI **Lint**).

---

## Environment setup

### Go

- **Go 1.26** (see `go.mod`)
- **xcaddy**: `go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest`

For building Caddy with caddy-snake:

```bash
xcaddy build --with github.com/mliezun/caddy-snake=.
```

### Python

- **Python 3.12+** (3.13 recommended)
- Each integration test app has its own `tests/<app>/requirements.txt`

**Flask integration** (`tests/flask`):

```bash
cd tests/flask
python3.13 -m venv venv
source venv/bin/activate   # or venv\Scripts\activate on Windows
pip install -r requirements.txt
```

**FastAPI integration** (`tests/fastapi`):

```bash
cd tests/fastapi
python3.13 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**Python tests** (root-level `caddysnake_test.py`):

```bash
# From project root
pip install -r requirements-dev.txt   # pytest, pytest-cov, pytest-asyncio
# Run from project root so caddysnake and tests.test_apps are importable
```

### Integration tests (Docker)

For full CI-like integration tests without local Python/venv setup:

```bash
./tests/integration.sh <tool-name> <python-version>
# Examples:
./tests/integration.sh flask 3.13
./tests/integration.sh fastapi 3.13
```

Valid tools: `django`, `django_channels`, `flask`, `fastapi`, `simple_autoreload`, `simple_async`, `simple_esgi`, `simple_cache`, `socketio`, `dynamic`
Valid Python versions: `3.12`, `3.13`, `3.13-nogil`, `3.14`, `3.14-nogil`

Requires **Docker** (linux/amd64 container).

---

## Running tests

### Go tests

```bash
# Full suite with race detector (recommended before commit)
go test -race -v .

# Quick run without race detector
go test -v .

# With coverage
go test -race -coverprofile=coverage.out .
go tool cover -html=coverage.out

# In-process Caddy integration (caddytest build tag — requires Python)
go test -race -tags=caddytest -timeout 180s .
```

### caddytest (in-process)

Use the **`caddytest`** build tag for **`caddysnake_caddytest_test.go`** (includes **HTTPS / on-demand TLS** coverage). Requires **`python`** on **`PATH`** and a generous timeout (**180s** is safe on CI-arm).

```bash
go test -race -tags=caddytest -timeout 180s .
```

### Python tests

```bash
# From project root (so caddysnake and tests.test_apps are on PYTHONPATH)
pytest caddysnake_test.py -v

# With coverage
pytest caddysnake_test.py -v --cov=caddysnake --cov-report=term-missing

# With verbose output and stop on first failure
pytest caddysnake_test.py -vx
```

### Integration tests (Flask, FastAPI, etc.)

**Option A — Docker (recommended for CI parity):**

```bash
./tests/integration.sh flask 3.13
./tests/integration.sh fastapi 3.13
```

**Option B — Local (faster feedback):**

```bash
# 1. Set up venv and build Caddy (once per app)
cd tests/flask
python3.13 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
CGO_ENABLED=0 xcaddy build --with github.com/mliezun/caddy-snake=../..

# 2. Start Caddy
./caddy run --config Caddyfile > caddy.log 2>&1 &

# 3. Wait for Caddy to be ready
timeout 60 bash -c 'while ! grep -q "finished cleaning storage units" caddy.log; do sleep 1; done'

# 4. Run integration test
source venv/bin/activate
python main_test.py


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mliezun/caddy-snake](https://github.com/mliezun/caddy-snake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
