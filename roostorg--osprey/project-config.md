---
trigger: always_on
description: Instructions for AI coding agents working on Osprey. `README.md` is for humans; this file is for machines. The nearest `AGENTS.md` to the edited file wins; explicit user prompts override everything.
---

# AGENTS.md

Instructions for AI coding agents working on Osprey. `README.md` is for humans; this file is for machines. The nearest `AGENTS.md` to the edited file wins; explicit user prompts override everything.

## Architecture

Top-level modules:

- `osprey_worker/` — main Python engine. Consumes events from Kafka, evaluates SML rules, emits verdicts and effects to output sinks. New worker/engine code belongs here (`osprey_worker/src/osprey/worker/`).
- `osprey_rpc/` — generated protobuf/gRPC bindings under `osprey_rpc/src/osprey/rpc/`. Do not edit generated files (`*_pb2*.py`, `*_pb2*.pyi`) by hand; regenerate via `./gen-protos.sh` after editing the `.proto` files.
- `osprey_ui/` — React + TypeScript frontend (Ant Design, Highcharts; versions in `osprey_ui/package.json`). UI code belongs here.
- `osprey_coordinator/` — Rust gRPC coordinator (tokio, tonic, etcd, rdkafka). Rust code belongs here.
- `proto/osprey/rpc/` — protobuf source of truth for `osprey_rpc` and `osprey_coordinator` types.
- `example_plugins/` — reference plugins (UDFs, output sinks, labels service) using the pluggy-based plugin system. Do not add production code here.
- `example_rules/` — sample SML rules and YAML config.

Reference files: `docs/DEVELOPMENT.md` (setup), `example_plugins/src/register_plugins.py` (plugin patterns), `example_plugins/src/services/labels_service.py` (labels service example).

## Design

- API: gRPC between `osprey_coordinator` and workers; HTTP/Flask for `osprey-ui-api` (port 5004); protobuf definitions under `proto/osprey/rpc/` are authoritative.
- Rules: SML (Osprey's rule language) with user-defined functions registered via pluggy hooks (`@hookimpl_osprey`): `register_udfs`, `register_output_sinks`, `register_labels_service_or_provider`.
- Data model conventions: Pydantic for models, SQLAlchemy for persistence (versions pinned in `pyproject.toml`).

## Build and run

Prerequisites: Python (version in `.python-version`), [uv](https://docs.astral.sh/uv/), Docker + Docker Compose v2, Node.js (version in `.github/workflows/code-quality.yml`, UI only), Rust stable + `protoc` (coordinator only).

```bash
# Install Python deps (creates .venv, uses uv.lock)
uv sync --dev

# Install git hooks
uv run pre-commit install --install-hooks

# Start full stack (Kafka, Postgres, Druid, MinIO, Bigtable emulator, worker, UI, UI API)
docker compose up -d
# or
./start.sh
# with coordinator:
./start.sh --with-coordinator

# UI dev server
cd osprey_ui && npm ci && npm start

# Regenerate protobuf bindings after editing proto/osprey/rpc/**/*.proto
./gen-protos.sh
```

UI: http://localhost:5002 · UI API: http://localhost:5004 · Worker (port 5001)

## Testing

Run the full integration suite (spins up all services via docker compose; ~8 GB RAM):

```bash
./run-tests.sh
```

Pass pytest args through:

```bash
./run-tests.sh path/to/test_file.py::test_name
./run-tests.sh -k some_keyword
./run-tests.sh --junitxml=/tmp/test-results/junit-pytest.xml
```

Python lint / format / type-check (no Docker needed):

```bash
uv run ruff check
uv run ruff format --diff
uv run mypy .
uv run pre-commit run --all-files
```

UI checks (in `osprey_ui/`):

```bash
npm run format:check
```

Rust checks (in `osprey_coordinator/`; requires `protoc`). CI only gates on `fmt` and `build`; `clippy` and `test` are advisory (`continue-on-error: true`):

```bash
cargo fmt --check
cargo build --verbose
cargo clippy -- -D warnings   # advisory
cargo test --verbose          # advisory
```

## CI

CI runs entirely via GitHub Actions on `pull_request` and `push` to `main`. Each line below is one literal CI `run:` step, in workflow order. Run them in your shell (paste-as-is — no `&&` chaining, no error suppression — so each step's exit code matches the corresponding CI step's exit code):

```bash
# code-quality.yml → python-quality
uv sync --dev
uv run pre-commit install --install-hooks
SKIP=prettier-osprey-ui uv run pre-commit run --show-diff-on-failure --color=always --all-files
uv tool run fawltydeps --check-unused --pyenv .venv

# code-quality.yml → ui-quality (CI `working-directory: osprey_ui`)
( cd osprey_ui
  npm ci
  npm run format:check )

# code-quality.yml → rust-quality (CI `working-directory: osprey_coordinator`)
# Note: in CI the `cargo clippy` and `cargo test` steps are marked `continue-on-error: true`,
# so they currently print failures but do not fail the job. Locally, expect the same output.
( cd osprey_coordinator
  cargo fmt --check
  cargo clippy -- -D warnings
  cargo build --verbose
  cargo test --verbose )

# integration-tests.yml
./run-tests.sh
```

`mdbook.yml`, `publish-coordinator-image.yml`, and `release-osprey-rpc.yml` are release/deploy workflows — do not modify without human approval (see "Human-approval-required actions" below).

## Security

- No secrets in code or committed files. Use environment variables via `docker-compose.yaml`.
- Do not disable lint or type rules to silence errors. Fix the underlying issue, or use a narrowly-scoped `# noqa: <code>` / `# type: ignore[<code>]` with a comment explaining why.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roostorg/osprey](https://github.com/roostorg/osprey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
