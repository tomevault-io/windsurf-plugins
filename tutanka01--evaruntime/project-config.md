---
trigger: always_on
description: Instructions for coding agents working on EVARuntime. This file applies to the
---

# AGENTS.md

Instructions for coding agents working on EVARuntime. This file applies to the
whole repository unless a more specific `AGENTS.md` is added in a subdirectory.

## Project Mission

EVARuntime is an OpenAI-compatible inference gateway for private GPU
infrastructure. It is built to keep prompts, model execution, access control,
usage logs, and operational controls inside the host organization.

The codebase favors pragmatic operations over a heavy platform stack:
FastAPI, SQLite WAL, systemd, nginx, and `llama.cpp` managed as gateway-owned
subprocesses.

## Repository Map

- `gateway/`: main OpenAI-compatible gateway.
- `gateway/cluster/`: multi-node orchestration, node scheduling, node client,
  and shared protocol models.
- `gateway/readiness.py`: structural readiness checks. Consumed by `/ready` and
  by `doctor`. Reuse it, do not reimplement its checks.
- `gateway/doctor.py`: host preflight (secrets, runtime, GPU, models, ports,
  nginx, TLS, systemd). Human and JSON output, stable exit codes.
- `gateway/deploy/`: systemd, nginx, installer, update script, and example node
  configuration.
- `gateway/deploy/smoke_test.sh`: first-token recipe. Gates `update.sh` and is
  runnable standalone during an incident.
- `gateway/deploy-macos/`: launchd/Homebrew mirror of `deploy/`. `deploy/` is
  the reference tree: parity is enforced by
  `gateway/tests/test_deploy_trees_parity.py` (issue #29).
- `gateway/static/dashboard.html`: admin dashboard served by the main gateway.
- `gateway/tests/`: main gateway unit tests.
- `node_agent/`: lightweight FastAPI agent that loads/unloads `llama-server` on a
  remote GPU node.
- `docs/`: main gateway architecture, API, admin, deployment, and research notes.

## Core Architecture

The main gateway exposes OpenAI-compatible routes and owns the model lifecycle.
Requests are authenticated, rate-limited, routed by `model`, and proxied to the
right `llama-server` backend. In local mode, backends are subprocesses launched
by `gateway/server_manager.py`. In cluster mode, `gateway/model_manager.py`
selects `ClusterManager`, which delegates model placement and lifecycle calls to
remote `node_agent` services.

The model registry is the source of truth for available models. Treat
`models.yaml` and `MODELS_CONFIG_PATH` as operational configuration, not casual
test data. Model IDs, file paths, VRAM estimates, capabilities, and per-model
`llama.cpp` parameters are validated by `gateway/model_registry.py`.

## Development Commands

Use Python 3.11 or newer.

Main gateway:

```bash
cd gateway
python -m venv .venv
source .venv/bin/activate
python -m pip install --require-hashes -r requirements.lock -r ../requirements-dev.lock
python -m pytest tests -v
uvicorn main:app --host 127.0.0.1 --port 8000 --reload
```

Node agent smoke setup:

```bash
cd node_agent
python -m venv .venv
source .venv/bin/activate
python -m pip install --require-hashes -r requirements.lock
uvicorn main:app --host 127.0.0.1 --port 9443 --reload
```

Do not run `gateway/tests` and `node_agent/tests` in one pytest process unless
you have verified import isolation. Both components intentionally use top-level
module names such as `config`, `database`, `auth`, and `schemas`.

### CI parity

The `.venv/` directories already exist and have `ruff` installed. Do not
recreate them; call them directly, e.g. `gateway/.venv/bin/python -m pytest tests -q`.

CI runs `ruff check .` per component **in addition to** pytest. Run it before you
call a change done, otherwise the failure only surfaces after the PR is opened.

CI runs on **Python 3.11** from the checked-in hashed lockfiles, while the local
venvs may be on 3.14 and retain older packages. To reproduce the CI environment:

```bash
uv venv --python 3.11 /tmp/ci
uv pip install --python /tmp/ci/bin/python --require-hashes \
  -r gateway/requirements.lock -r requirements-dev.lock
cd gateway && /tmp/ci/bin/python -m pytest tests -q
```

## Coding Style

- Match the existing Python style: typed functions where useful, direct module
  functions/classes, and small FastAPI handlers that delegate to focused modules.
- Keep async code async end to end. Do not introduce blocking subprocess, network,
  or file operations into request paths without pushing them out of the hot path.
- Keep comments concise and useful. The existing code uses French comments and
  operational docstrings; follow that style when editing nearby code.
- Prefer standard-library and existing dependency solutions. Avoid adding new
  dependencies unless they remove real operational risk or substantial complexity.
- Preserve OpenAI-compatible response shapes and error bodies on `/v1/*` routes.
- Keep deployment files boring and explicit. systemd/nginx scripts are production
  artifacts, not generated scratch files.
- A list-typed setting loaded from the environment must be annotated
  `str | list[str]` and normalized by a `mode="before"` validator
  (`split_list_setting`). Annotated as a bare `list[str]`, pydantic-settings
  decodes it as JSON **in the source, before any validator runs**: the documented
  comma-separated syntax — and even an empty value — fail startup with
  `SettingsError`, and the validator never executes. This shipped broken for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tutanka01/EVARuntime](https://github.com/Tutanka01/EVARuntime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
