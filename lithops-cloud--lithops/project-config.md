---
trigger: always_on
description: Instructions for AI coding agents working in this repository (OpenAI Codex, GitHub Copilot
---

# AGENTS.md

Instructions for AI coding agents working in this repository (OpenAI Codex, GitHub Copilot
coding agent, Cursor, Gemini CLI, Aider, Claude Code and others). Human contributors should
read [CONTRIBUTING.md](CONTRIBUTING.md); this file summarizes the same rules plus the
architectural context an agent needs.

## Project overview

Lithops is a Python framework that runs unmodified Python functions at scale on serverless
platforms, container and batch services, virtual machines and the local machine, behind one
small API (`FunctionExecutor.call_async()`, `map()`, `map_reduce()`, `wait()`,
`get_result()`, `Storage`). The architecture is described in
[docs/source/design.rst](docs/source/design.rst); read it before changing core code.

- **Client:** where `import lithops` runs. Builds jobs, serializes code and data, invokes the
  workers and collects results. All orchestration lives here.
- **Compute backend:** where the workers run, one call each.
- **Object storage:** the only communication bus between client and workers. The client
  writes the pickled function and data; workers read their input and write their output and
  status back. There are no direct client-worker connections.

## Setup and commands

```bash
pip3 install -e '.[tests,dev]'             # editable install + test deps + ruff / pre-commit
                                           # (backend extras: '.[aws]', '.[gcp]', '.[all]', ...)
ruff check .                               # lint; must pass (config in pyproject.toml, line length 120)

cd lithops/tests
pytest -v --timeout=120 --timeout-method=thread --backend localhost --storage localhost
pytest -v --backend localhost --storage localhost -k test_map    # a subset
pytest --collect-only                                            # list the tests
```

**Always pass `--backend localhost --storage localhost`.** Without them the test session
loads the developer's own configuration (`~/.lithops/config`, `.lithops_config`,
`LITHOPS_CONFIG_FILE`) and runs against whatever cloud account it points to. Some tests need a
Redis server on `localhost:6379` and skip themselves when none is reachable.

Documentation is built with Sphinx from `docs/` (`pip3 install -e '.[docs]'`, then
`make -C docs html SPHINXOPTS="-W --keep-going"`, see [docs/README.md](docs/README.md)). Pull
requests that touch the docs must build without warnings: CI runs that same command.

## Repository map

| Path | Content |
|---|---|
| `lithops/__init__.py` | Public API surface (`FunctionExecutor`, `Storage`, `wait`, `get_result`, ...) |
| `lithops/executors.py` | `FunctionExecutor` and `LocalhostExecutor` / `ServerlessExecutor` / `StandaloneExecutor` |
| `lithops/config.py` | Loads and merges configuration (dict, env vars, YAML), validates backends |
| `lithops/job/` | Job creation, function/module serialization (`cloudpickle`), data partitioning |
| `lithops/invokers.py` | `FaaSInvoker` (per-call invocations) and `BatchInvoker` (one submission, many tasks) |
| `lithops/future.py`, `wait.py`, `retries.py` | `ResponseFuture`, `wait()` / `get_result()`, `RetryingFunctionExecutor` |
| `lithops/worker/` | Code that runs on the compute backend: `handler.py` (`function_handler`) and `jobrunner.py` (`JobRunner`) |
| `lithops/serverless/backends/<name>/` | Serverless / batch compute backends (`<name>.py`, `config.py`, `entry_point.py`) |
| `lithops/standalone/` | Standalone mode: master / worker VMs coordinated through Redis; `backends/<name>/` for VM providers |
| `lithops/localhost/` | Localhost compute backend (`v1`, `v2`) |
| `lithops/storage/` | `Storage` / `InternalStorage`, cloud file APIs; `backends/<name>/` for object stores |
| `lithops/monitoring/` | Job monitor and its pluggable backends (storage polling, RabbitMQ, Redis, queues) |
| `lithops/telemetry/` | Prometheus / OpenTelemetry metrics (off by default) |
| `lithops/multiprocessing/`, `concurrent/`, `util/joblib/` | Drop-in `multiprocessing`, `concurrent.futures` and joblib APIs |
| `lithops/scripts/` | `lithops` CLI (`cli.py`) and the temporary-data cleaner |
| `lithops/tests/` | pytest suite (`conftest.py` defines `--backend`, `--storage`, `--config`, `--region`) |
| `runtime/<backend>/` | Dockerfiles and instructions to build runtimes for each backend |
| `config/` | `config_template.yaml` (every configuration key) and the configuration guide |
| `docs/` | Sphinx documentation; `docs/source/compute_config/` and `storage_config/` per backend |
| `examples/` | Single-file usage examples |

## Architectural invariants

1. **Storage is the bus.** Workers receive their input and return their results and status
   through object storage (`func_key`, `agg_data`, `output_key`, `status_key`), never through
   a direct connection to the client.
2. **Every backend looks the same to the core.** Backend-specific code stays in its backend
   package; the executor, invoker, job and worker code must not special-case a backend by
   name when an interface method or a config value can express it.
3. **The worker must tolerate the environment it runs in.** Worker code runs in minimal
   runtimes (Lambda, containers, VMs, macOS/Windows localhost). Imports of optional

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lithops-cloud/lithops](https://github.com/lithops-cloud/lithops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
