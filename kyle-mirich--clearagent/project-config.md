---
trigger: always_on
description: ClearAgent is an installable Python 3.14 library. Use `uv`, preserve the
---

# Agent Instructions

## Fast Orientation

ClearAgent is an installable Python 3.14 library. Use `uv`, preserve the
local-first SQLite default, and keep the default test path offline and
deterministic.

Bootstrap a checkout without changing the lockfile:

```bash
uv sync --locked --all-extras --dev
```

While iterating, run the smallest checks that cover the change:

```bash
uv run pytest tests/unit/test_tool_schema.py
uv run pytest tests/integration/test_agent_tracing.py
uv run ruff check src/clearagent/tool.py tests/unit/test_tool_schema.py
uv run python -m mypy src
uv run python scripts/check_docs_links.py
```

Before handoff, run the full offline gate when practical:

```bash
./scripts/check.sh
```

The gate runs offline tests with at least 90% package line coverage, Ruff lint
and formatting checks, mypy, and the documentation checker. Run `uv build` as
an additional check for package metadata, bundled files, entry points, or
release workflow changes.

## Repository Map

Use this map to find the implementation, its focused verification, and the
reader-facing docs that normally move with it.

| Task | Implementation | Focused tests | Public docs |
| --- | --- | --- | --- |
| Agent runtime, tools, messages, and results | `src/clearagent/agent.py`, `create.py`, `tool.py`, `messages.py`, `types.py` | `tests/unit/test_create_agent.py`, `test_tool_schema.py`, `test_structured_output.py`; `tests/integration/test_agent_tracing.py` | `docs/getting-started.md`, `core-concepts.md`, `reference.md` |
| Provider adapters and model URIs | `src/clearagent/providers/` | `tests/unit/test_*provider.py`, `test_model_uri.py`, `test_native_providers.py`, `test_live_provider_compatibility.py` | `docs/providers.md`, `live-provider-compatibility.md`, `reference.md`, `status.md` |
| Trace storage, replay, and reports | `src/clearagent/storage/`, `trace_lifecycle.py`, `replay.py`, `reports.py` | `tests/unit/test_sqlite_trace_store.py`, `test_replay.py`, `test_reports.py`; `tests/integration/test_trace_cli.py`, `test_custom_trace_store.py` | `docs/tracing.md`, `database.md`, `architecture.md`, `reference.md` |
| Eval suites, checks, baselines, and Promptfoo | `src/clearagent/evals/` | `tests/unit/test_eval_*.py`, `test_promptfoo_export.py`; `tests/integration/test_eval_*.py`, `test_baselines.py` | `docs/evals.md`, `pytest.md`, `promptfoo.md`, `reference.md` |
| Graph execution | `src/clearagent/graph/` | `tests/integration/test_agent_graph.py` | `docs/core-concepts.md`, `flows.md`, `architecture.md`, `reference.md` |
| Chat backend and browser assets | `src/clearagent/chat/` | `tests/unit/test_chat_store.py`; `tests/integration/test_chat_app.py` | `docs/chat.md`, `database.md`, `flows.md`, `reference.md` |
| CLI and project config | `src/clearagent/cli.py`, `config.py` | `tests/unit/test_config.py`; `tests/integration/test_cli_smoke.py`, `test_cli_json.py`, `test_trace_cli.py`, `test_baselines.py` | `docs/reference.md` and the relevant workflow guide |
| Pytest integration | `src/clearagent/pytest_plugin/` | `tests/integration/test_pytest_integration.py` | `docs/pytest.md`, `core-concepts.md`, `reference.md` |
| Packaging, examples, and docs | `pyproject.toml`, `examples/`, `scripts/`, `.github/workflows/`, `README.md`, `docs/` | `tests/unit/test_public_api.py`, `test_docs_links.py`; `tests/integration/test_examples.py`; `uv build` | `docs/install.md`, `publishing.md`, `deployment.md`, `contributing-docs.md` |

## Core Invariants

- A provider-shaped request is built and redacted before it is persisted, and
  it is persisted before the provider call. Preserve this order so failures can
  still be inspected and requests can be replayed exactly.
- `TraceStore` is the runtime, eval, trace-check, graph, and agent-backed chat
  persistence boundary. An injected store must not be silently replaced by a
  newly opened SQLite store. Standalone file-oriented CLI inspection remains
  explicitly SQLite-backed through `--trace-db`.
- SQLite is the default, not an assumption for injected-store code. Keep trace
  and chat persistence separate.
- Agent tool loops are bounded by `max_turns`; graph runs are linear, reject
  cycles and unknown targets, and are bounded by `max_nodes`.
- The normal test gate never calls a live model. The bounded compatibility
  runner requires `CLEARAGENT_LIVE_TESTS=1` and the relevant provider
  credential; default tests consume sanitized recordings.
- Public examples import authoring helpers from `clearagent`, provider values
  from `clearagent.providers`, storage values from `clearagent.storage`, and
  other documented package entry points. Treat deeper implementation modules
  as internal unless `docs/reference.md` says otherwise.
- Chat binds to loopback by default. Runtime settings mutation stays disabled
  unless the caller explicitly enables it.

## Files And Commands That Write

Hand-authored source includes `src/`, `tests/`, `examples/`, curated Markdown,
browser assets, project metadata, the lockfile, scripts, and workflows. Treat
these outputs as generated until a person deliberately reviews and promotes
them:

- `.clearagent/*.sqlite`, plus SQLite `-wal` and `-shm` sidecars
- `.clearagent/promptfoo_target.py`, exported Promptfoo configs, replayed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kyle-mirich/clearagent](https://github.com/kyle-mirich/clearagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
