---
trigger: always_on
description: `agent/` contains the generation runtime, provider adapters, prompt compiler/loader, tools, cost tracking, TUI helpers, and single-record orchestration. `storage/` owns the gitignored local `data/` layout, records, categories, `records_manifest.jsonl`, run caches, and materialization metadata. `sdk/` and `sdk/_core/` define the articulated-object SDK layers; `sdk/_docs/` and `sdk/_examples/` are agent-facing authoring assets. `viewer/api/` exposes the FastAPI surface, and `viewer/web/` is the Re
---

# Repository Guidelines

## Project Structure & Module Organization
`agent/` contains the generation runtime, provider adapters, prompt compiler/loader, tools, cost tracking, TUI helpers, and single-record orchestration. `storage/` owns the gitignored local `data/` layout, records, categories, `records_manifest.jsonl`, run caches, and materialization metadata. `sdk/` and `sdk/_core/` define the articulated-object SDK layers; `sdk/_docs/` and `sdk/_examples/` are agent-facing authoring assets. `viewer/api/` exposes the FastAPI surface, and `viewer/web/` is the React/TypeScript/Three.js viewer. `cli/` contains the `articraft` entry points. `tests/` mirrors the main packages with focused smoke/regression coverage.

## Build, Test, and Development Commands
Use `uv run articraft ...` for product workflows, and `just` for local setup/check/viewer shortcuts. Run `just` to list available shortcuts.

- `uv sync --group dev` installs Python and development dependencies.
- `just setup` bootstraps `.env`, syncs dependencies, installs viewer dependencies when `npm` exists, and initializes storage.
- `uv build` creates the wheel and source distribution in `dist/`.
- `uv run articraft init` creates the gitignored local `data/` tree and empty library manifest.
- `uv run articraft status` shows local library status.
- `uv run --group dev pytest -q` runs the full Python regression suite.
- `just smoke-tests` runs the fast pre-push suite; `just test-all` runs the full Python suite.
- `just format` runs `ruff format .`; `just lint` runs `ruff check .`.
- `uv run articraft library check` validates local library format.
- `uv run articraft env bootstrap` creates `.env` from `.env.example` without overwriting existing secrets.

## Generation, Dataset, and Viewer Commands
- External agent data generation must follow [`EXTERNAL_AGENT_DATA.md`](EXTERNAL_AGENT_DATA.md). If the user asks Codex, Claude Code, Cursor, or another external harness to generate Articraft data, use `uv run articraft external ...`; do not manually create records or use an alternate workflow.
- `uv run articraft generate "prompt text"` writes a local library record.
- `uv run articraft generate --model gemini-3-flash-preview --image reference.png "prompt text"` overrides model and adds a reference image.
- `uv run articraft draft "prompt text"` creates a draft local record without running generation.
- `uv run articraft draft --image reference.png "prompt text"` stores a reference image with the draft.
- `uv run articraft rerun <record-id>` reruns generation for an existing record.
- `uv run articraft compile <record-id>` recompiles one record into `<data-root>/cache/record_materialization/<id>/`.
- `uv run articraft library list` lists records from `records_manifest.jsonl`.
- `uv run articraft library rebuild-manifest` rebuilds `records_manifest.jsonl` from `records/`.
- `uv run articraft library check --require-records` validates that manifest rows point at real records.
- `just viewer` starts the built viewer flow; `just viewer-dev` starts uvicorn and Vite together.
- `uv run uvicorn viewer.api.app:app --reload --host 127.0.0.1 --port 8765` starts only the API.
- `npm --prefix viewer/web run dev`, `build`, `lint`, and `typecheck` run frontend-only workflows.

## Coding Style & Naming Conventions
Target Python 3.11+; `.python-version` pins 3.12 for local `uv` use and the project excludes Python 3.13. Keep 4-space indentation, `from __future__ import annotations`, explicit type hints, and small module-level helpers for CLI wiring. Python formatting and import checks are handled by Ruff (`line-length = 100`, target `py311`, rules `E`, `F`, `I`, ignoring `E501`). Use `snake_case` for functions/modules/variables and `PascalCase` for models/classes. For `viewer/web`, use strict TypeScript, ESLint, Tailwind CSS v4, shadcn/ui components, and the `@/` import alias.

## Testing Guidelines
Tests run under `pytest` with importlib mode and xdist auto/worksteal by default. Add coverage under the mirrored package path, name files `test_<feature>.py`, and prefer native pytest functions and fixtures with plain `assert` statements. Keep coverage focused on fast import, storage, CLI, viewer API, SDK, and integration smoke checks. For prompt regressions, prefer durable behavioral checks over brittle formatting or line-budget assertions; relax or remove stale prompt assertions in the same change that updates the intended prompt contract.

## Commit & Pull Request Guidelines

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mattzh72/articraft](https://github.com/mattzh72/articraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
