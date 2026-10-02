---
trigger: always_on
description: Guidance for coding agents and contributors working in this repository.
---

# AGENTS.md

Guidance for coding agents and contributors working in this repository.

## What agent-env is

agent-env is a Python SDK for building environments that AI agents act in, and for running and
evaluating agents against them. It deploys environments (MCP servers, databases, websites, gateways)
into sandboxes, keeps immutable versioned artifacts, runs tasks as DAGs of steps (deploy an env,
deploy an agent, prompt it, verify the outcome) and records every run. Persistence, sandboxes, the
model endpoint and every extension are selected in one config file; a bare install runs fully local.
The in-repo `packages/agentenv-protocol` is the open data-plane protocol and server SDK for
environments; agent-env depends on it.

## Setup

Python 3.11 or newer. The repo convention is a `.venv` at the root, which the Makefile and CI use.

```bash
uv sync --extra dev && . .venv/bin/activate
```

Without uv:

```bash
python3.11 -m venv .venv && . .venv/bin/activate
pip install -e ./packages/agentenv-protocol -e '.[dev]'
```

Both install the in-repo protocol package together with the dev extra: agent-env depends on it and
the workspace copy is the one to develop against (`uv sync` does this through the uv workspace in
`pyproject.toml`). `make install` is the pip route in one step. No cloud credentials are needed; the
default stores are local.

## Commands

| Command | What it does |
|---|---|
| `make unit-test` | `tst/unit` and `packages/agentenv-protocol/tests`, in parallel, no network |
| `make int-test-fast` | `tst/integration` minus `int_test_slow`, in parallel; needs Docker and a local OCI registry (`docker run -d -p 5000:5000 public.ecr.aws/docker/library/registry:2`) |
| `make int-test-slow` | the `int_test_slow` tests, serially (they build the same Docker tags from module fixtures and race under xdist) |
| `make installer-test` | `tst/installer`: `plugin add` / `remove` through the real pip, uv and pipx, offline against wheels built from the checkout, and the container user journey (`python:3.12-slim`, `--network none`) with the PEP 668 refusal; needs uv, pipx and Docker |
| `make clean-install-test` | the `clean-install` CI gate: both distributions built as the release builds them, installed into a fresh venv from public PyPI with an allowlisted environment, and `agent-env run hello` run twice by name and checked through the local store; needs Python 3.11, uv and git |
| `pytest tst/unit/env/store_test.py -v` | one file; add `--log-cli-level=DEBUG` for debug logs |
| `uv build` | wheel `agentenv_framework-<version>-py3-none-any.whl` (`src/agent_env` only) and sdist `agentenv_framework-<version>.tar.gz` (`src`, `tst`, the protocol package) |
| `agent-env`, `python -m agent_env.cli` | the CLI; `agent-env config show` prints which config file is in effect and where each section came from |

## Configuration

- One file: `.agentenv/config.toml`, found by walking up from the working directory, or the file
  `AGENT_ENV_CONFIG` names (it must exist). Exactly one file is read, whole; a section that is absent
  falls back to the code default, never to another file. `.agentenv/config.example.toml` is the
  committed template; `.agentenv/config.toml` is git-ignored.
- Precedence per section: built-in default < config file < `AGENT_ENV_*` environment variable
  (`AGENT_ENV_DOCUMENT_STORE`, `AGENT_ENV_OBJECT_STORE`, `AGENT_ENV_IMAGE_STORE`,
  `AGENT_ENV_SECRET_STORE`, `AGENT_ENV_RUNNER`) < `configure(...)` in code.
- Defaults are local and carry no external coordinates: SQLite document store, filesystem object
  store, an OCI registry at `localhost:5000`, env-var secret store, `LocalRunner`, `local` Docker
  sandboxes for envs and agents. MongoDB, S3, Cloud Storage, ECR, AWS Secrets Manager, Google Cloud
  Secret Manager, Modal and E2B exist as implementations and are selected by config.
- The model endpoint is unset until `[model] base_url` / `api_key` (or `LITELLM_BASE_URL` /
  `LITELLM_API_KEY`) is configured.
- `[agents] default_a2a_agent_id`: the agent a `deploy_agent` step without an id deploys; built-in
  `a2a-default`, overridable by `configure(default_a2a_agent_id=...)`.
- Secrets never live in the file: use `secret:KEY` or `env:NAME` references, resolved through
  `[stores.secret]`.
- agent-env has no stage concept: "dev" versus "prod" is which file `AGENT_ENV_CONFIG` names. A
  platform plugin may pick that file; core must never read a stage variable, take a stage argument or
  carry a stage attribute (`tst/unit/config/test_no_stage_concept.py` enforces this).
- Re-pointing `AGENT_ENV_CONFIG` after the first store or registry is built is unsupported: set it
  before anything resolves, or call `reset_config()`, which drops the singleton and every registry.

## Architecture

Source lives in `src/agent_env/`. Every kind of object is identified on the wire by a `type` string
and deserialized through a registry.

| Package | Contents |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scaleapi/agentenv-framework](https://github.com/scaleapi/agentenv-framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
