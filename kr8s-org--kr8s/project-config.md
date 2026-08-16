---
trigger: always_on
description: Guidance for AI agents (and humans) working in the `kr8s` repository.
---

# AGENTS.md

Guidance for AI agents (and humans) working in the `kr8s` repository.

`kr8s` is a simple, extensible Python client library for Kubernetes that feels
familiar to people who already know `kubectl`. It ships a synchronous API by
default and an async API (`asyncio`/`trio`) under `kr8s.asyncio`.

## Repository layout

- `kr8s/` — the library source.
  - `_objects.py` — the heart of the project. Defines `APIObject` (async) and
    every Kubernetes resource class (`Pod`, `Deployment`, `Service`, …). All
    real logic lives here as async methods named with an `async_` prefix
    (e.g. `async_get`, `async_create`, `async_delete`).
  - `_api.py` — the async `Api` client (auth, request dispatch, caching).
  - `_auth.py`, `_config.py` — kubeconfig loading, auth, and cluster config.
  - `_async_utils.py` — the machinery that lets the sync API wrap async code by
    running a background thread with its own event loop (`Portal`,
    `as_sync_func`, `as_sync_generator`). Rarely needs changing.
  - `objects.py`, `_api.py`-derived sync surfaces — the **sync** public API.
    `APIObjectSyncMixin` (in `_objects.py`) and `kr8s/objects.py` wrap the
    `async_*` methods with `as_sync_func` / `as_sync_generator`.
  - `asyncio/` — the **async** public API. Thin re-exports of the underlying
    async classes/functions from `kr8s._*` modules.
  - `_exec.py`, `_portforward.py`, `_data_utils.py`, `_exceptions.py` — helpers.
  - `_vendored/` — third-party code (e.g. `asyncache`). **Do not lint, format,
    or edit** — it is excluded from all tooling.
  - `tests/` — pytest suite (`test_*.py`), plus `resources/` and `scripts/`.
  - `conftest.py` (both root and `kr8s/`) — shared fixtures.
- `docs/` — Sphinx documentation (MyST markdown). `contributing.md` and
  `releasing.md` are the canonical process docs.
- `examples/kubectl-ng/` — a separate `kubectl-ng` package (a `kubectl` clone
  built on kr8s) with its **own** `pyproject.toml` and test suite.
- `ci/` — automation scripts (e.g. `update-kubernetes.py`).
- `.github/workflows/` — CI (`test-kr8s.yaml`, `test-kubectl-ng.yaml`, release).

## The sync/async architecture (read this before editing resource code)

kr8s is built async-first. **All real implementation is async and lives once.**

1. Write logic as an `async def async_<name>(...)` method on the async class in
   `kr8s/_objects.py` (or `_api.py`).
2. Expose it on the async API via `kr8s/asyncio/` re-exports (no wrapping — the
   async method is the public async method minus the `async_` prefix where the
   public name is defined, or exposed directly).
3. Expose it on the sync API by adding a wrapper in the `*SyncMixin` /
   `kr8s/objects.py` that calls `as_sync_func(self.async_<name>)(...)` (or
   `as_sync_generator` for generators).

When you add or change a method, update **both** the async source and its sync
wrapper, and keep signatures in sync. Sync wrappers carry
`# type: ignore[override]` because they intentionally override async signatures
with sync ones — this is expected. Docstrings live on the async version; sync
wrappers inherit them (`kr8s/objects.py` sets `# ruff: noqa: D102`).

## Development environment

Use [`uv`](https://github.com/astral-sh/uv) (the project's chosen tool) but
anything that installs the package in editable mode works.

```bash
pip install uv
uv sync --dev
```

Work on branches in git worktrees rather than switching branches in place.
Create worktrees under a `.worktrees/` directory in the project root
(e.g. `git worktree add .worktrees/my-feature -b my-feature`). This directory is
gitignored so worktrees are never tracked in version control.

Common tasks are defined via [`taskipy`](https://github.com/taskipy/taskipy) in
`pyproject.toml`:

- `uv run task test` — run the test suite.
- `uv run task test-ci` — tests with retries (what CI runs).
- `uv run task docs` — build docs once.
- `uv run task docs-serve` — live-reloading docs server.

## Testing

Tests run with `pytest` and use [`kind`](https://kind.sigs.k8s.io/) via
[`pytest-kind`](https://pypi.org/project/pytest-kind/) to spin up a real
Kubernetes cluster in Docker. **Docker must be available.** Cluster lifecycle is
handled by fixtures, so no manual cluster setup is needed.

```bash
uv run task test
# or a single file / test
uv run pytest kr8s/tests/test_objects.py
uv run pytest kr8s/tests/test_objects.py::test_pod_create
```

Key testing facts:

- Tests are **integration-heavy** and hit a live kind cluster; they are slower
  than pure unit tests. A per-test timeout of 300s is configured.
- `pytest-asyncio` runs in `asyncio_mode = "auto"`, so `async def test_*`
  functions work without decorators.
- Each test gets a fresh namespace via the autouse `ns` fixture
  (`kr8s/conftest.py`); use it to avoid cross-test collisions.
- Useful fixtures: `example_pod_spec`, `example_deployment_spec`,
  `example_service_spec`, `example_crd_spec`, `serviceaccount`, `k8s_cluster`.
- Prefer standard-library `unittest.mock` (`Mock`, `AsyncMock`, `patch`, and
  `patch.object`) for test doubles and temporary replacements. Avoid pytest's
  `monkeypatch` when `unittest.mock` can express the same behavior.
- Set `KUBERNETES_VERSION` to test against a specific k8s version. CI runs a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kr8s-org/kr8s](https://github.com/kr8s-org/kr8s) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
