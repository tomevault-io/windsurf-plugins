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
- `osprey_ui/` — React + TypeScript frontend (Ant Design, ECharts; versions in `osprey_ui/package.json`). UI code belongs here.
- `osprey_coordinator/` — Rust gRPC coordinator (tokio, tonic, etcd, rdkafka). Rust code belongs here.
- `proto/osprey/rpc/` — protobuf source of truth for `osprey_rpc` and `osprey_coordinator` types.
- `example_plugins/` — reference plugins (UDFs, output sinks, labels service) using the pluggy-based plugin system. Do not add production code here.
- `example_atproto_plugins/` — reference plugin demonstrating a custom input stream that consumes the Bluesky firehose. Stack `docker-compose.atproto.yaml` on top of the main compose file (or use `./run-atproto.sh`) to run Osprey against live ATProto traffic. Do not add production code here.
- `example_rules/` — sample SML rules and YAML config.
- `example_atproto_rules/` — sample SML rules paired with `example_atproto_plugins/`.

Reference files: `docs/development/local.md` (setup), `example_plugins/src/register_plugins.py` (plugin patterns), `example_plugins/src/services/labels_service.py` (labels service example).

## Design

- API: gRPC between `osprey_coordinator` and workers; HTTP/Flask for `osprey-ui-api` (port 5004); protobuf definitions under `proto/osprey/rpc/` are authoritative.
- Rules: SML (Osprey's rule language) with user-defined functions registered via pluggy hooks (`@hookimpl_osprey`): `register_udfs`, `register_output_sinks`, `register_labels_service_or_provider`, `register_input_stream` (custom event source; see `example_atproto_plugins/`).
- Data model conventions: Pydantic for models, SQLAlchemy for persistence (versions pinned in `pyproject.toml`).

## Build and run

Prerequisites: Python (version in `.python-version`), [uv](https://docs.astral.sh/uv/), Docker + Docker Compose v2, Node.js (version in `.github/workflows/code-quality.yml`, UI only), Rust stable + `protoc` (coordinator only).

```bash
# Install Python deps (creates .venv, uses uv.lock)
uv sync --dev

# Install git hooks
uv run pre-commit install --install-hooks

# Start full stack (Kafka, Postgres, Druid, MinIO, worker, UI, UI API)
docker compose up -d
# or
./start.sh
# with coordinator:
./start.sh --with-coordinator

# UI dev server (Corepack auto-resolves pnpm from osprey_ui/package.json's packageManager field)
cd osprey_ui && corepack enable && pnpm install --frozen-lockfile && pnpm start

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
pnpm run format:check
```

Rust checks (in `osprey_coordinator/`; requires `protoc`). CI only gates on `fmt` and `build`; `clippy` and `test` are advisory (`continue-on-error: true`):

```bash
cargo fmt --check
cargo build --verbose
cargo clippy -- -D warnings   # advisory
cargo test --verbose          # advisory
```

## Browser MCP (UI verification)

A project-scoped MCP server (`.mcp.json` at repo root) registers `@playwright/mcp@0.0.73` via `npx`. When Claude Code launches in this repo it gets `browser_navigate`, `browser_snapshot`, `browser_evaluate`, `browser_take_screenshot`, and the rest of the `browser_*` tool surface — useful for verifying visual UI changes against the running dev server without a full automated test suite. There is intentionally no `playwright.config.ts` / `@playwright/test` integration and no devDep in `osprey_ui/package.json`; the MCP is for ad-hoc verification, not CI.

All commands in this section — including any `--dry-run` previews — are **operator-run only**. Agents must never execute them. If a prereq is missing, the agent surfaces *what the operator should run* and waits.

The MCP needs Playwright's bundled Chromium binary plus its system shared libs. Setup is platform-specific — Playwright's docs cover it across macOS / Windows / WSL / Linux: <https://playwright.dev/docs/browsers#install-browsers>.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roostorg/osprey](https://github.com/roostorg/osprey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
