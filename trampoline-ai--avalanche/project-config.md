---
trigger: always_on
description: Avalanche is a local-first Python toolkit for defining and running data-flow DAGs. It combines a small decorator-based workflow API, Iceberg and Lance storage helpers, Local and Ray execution, an optional operator/gRPC control plane, and a Textual TUI. The project is an early release candidate: keep operational claims local-development focused and do not imply production auth, multi-tenancy, deployment, or durable recovery that does not exist.
---

# Repository Guidelines

## Project Overview

Avalanche is a local-first Python toolkit for defining and running data-flow DAGs. It combines a small decorator-based workflow API, Iceberg and Lance storage helpers, Local and Ray execution, an optional operator/gRPC control plane, and a Textual TUI. The project is an early release candidate: keep operational claims local-development focused and do not imply production auth, multi-tenancy, deployment, or durable recovery that does not exist.

## Architecture & Data Flow

1. **DAG authoring:** `src/avalanche/dag.py` captures decorated node calls inside a `ContextVar`-scoped `@workflow`. `NodeFuture`, `>>`, and `&` describe dependencies; workflow bodies should define graph edges rather than perform runtime work.
2. **Embedded execution:** `Workflow.run()` creates an awaitable `RunHandle` and starts a driver thread. The driver schedules nodes topologically, resolves annotated runtime providers and dependency references, submits work to `LocalExecutor` or `RayExecutor`, and fetches only final outputs. Payloads, status, lineage, and execution receipts remain separate channels.
3. **Runtime injection and storage:** providers in `src/avalanche/runtime/providers/` inject streams, models, logging, run input, and context. Backend-neutral contracts live in `src/avalanche/storage.py`; Iceberg and Lance implementations preserve snapshot and rerun lineage.
4. **Operator mode:** `src/runtime/operator/` discovers workflow descriptors, runs each workflow in a spawned coordinator process, and exchanges serializable lifecycle events over queues. The gRPC server exposes discovery, run control, logs, and update streams.
5. **TUI:** `GrpcStateProvider` implements the `StateProvider` boundary. `src/tui/ui_store.py` is the single mutable UI state owner; background threads enqueue guarded updates that the Textual UI applies on its own thread.

Preserve these boundaries. Do not pass live `Workflow` objects across operator or TUI process boundaries, collapse distributed references into eager local values, or bypass provider-based dependency injection.

## Key Directories

- `src/avalanche/`: public library API, DAG construction/execution, runtime models, storage contracts, Iceberg/Lance backends, and agent support.
- `src/runtime/`: executor implementations and the operator runtime, including discovery, scheduling, workers, gRPC, and generated protobuf modules.
- `src/tui/`: Textual application, widgets, DAG layout, mock provider, and `UIStore`.
- `src/ava_cli/`: the `ava` command-line entry point.
- `test/`: behavior and integration tests, with focused suites under `operator_tests/`, `storage/`, `iceberg/`, and `agent/`.
- `examples/`: smoke-tested executable workflows covering DAGs, streams, cursors, and operator discovery.
- `docs/`: architecture, API, execution-service, agent, and release guidance.
- `scripts/`: maintenance and performance tooling, including `benchmark_tui.py`.

## Development Commands

Use repository commands rather than ad hoc tool invocations:

```bash
uv sync --all-extras                 # install the complete development environment
uv run pre-commit install            # enable repository commit hooks once per clone
make lint                            # Ruff checks
make format                          # Ruff fixes and formatting
make test                            # full pytest suite with all extras
make precommit-check                 # lint + full tests; `make check` is an alias
make smoke-test                      # bounded documented-user-path tests
make test-cov                        # terminal branch coverage
make test-cov-html                   # HTML coverage report
make tui-bench                       # enforce TUI refresh budgets
make web-lint                       # ESLint checks for the browser UI
make web-format                     # format the browser UI with Prettier
make web-format-check               # check browser formatting without changes
make web-assets-check                # rebuild and reject stale packaged web assets
uv run pytest test/foo_test.py -q    # focused test while iterating
uv build                             # required after packaging/entry-point changes
```

Useful runtime examples:

```bash
uv run python examples/complex_dag_pattern.py
uv run ava dev --flows examples
uv run ava operator --flows examples --port 7433
uv run ava tui --connect localhost:7433
uv run ava tui                            # mock TUI
```

Avoid `--flows .`: operator discovery imports Python files recursively. Use a narrow file or directory.

## Code Conventions & Common Patterns

- Target Python 3.11–3.13. Use modern precise annotations, `Protocol`,
  dataclasses, and Pydantic models. Treat the declared domain schema as the only
  valid in-process shape.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Trampoline-AI/avalanche](https://github.com/Trampoline-AI/avalanche) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
