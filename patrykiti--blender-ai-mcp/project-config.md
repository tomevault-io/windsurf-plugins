---
trigger: always_on
description: `blender-ai-mcp` is a split Blender control system for LLMs:
---

# AGENTS.md

## Project Purpose

`blender-ai-mcp` is a split Blender control system for LLMs:

- `server/` exposes FastMCP tools and translates them to RPC calls.
- `blender_addon/` runs inside Blender and executes the actual `bpy` operations.
- `server/router/` is a supervisor layer that corrects, expands, and routes LLM tool calls through metadata and workflows.

The project exists to avoid raw-code Blender automation. The intended product surface is a stable, deterministic tool API with strong inspection, recovery, and workflow support.

## Repo Map

- `server/domain/`: abstract tool interfaces and core models. Keep framework-free.
- `server/application/tool_handlers/`: RPC-backed application handlers that implement domain interfaces.
- `server/adapters/mcp/areas/`: FastMCP tool definitions grouped by area.
- `server/adapters/rpc/`: socket RPC client used by handlers.
- `server/infrastructure/`: DI and config.
- `server/router/`: router, metadata, classifiers, workflow engine, vector store, and MCP integration helpers.
- `blender_addon/application/handlers/`: Blender-side handlers using `bpy`.
- `blender_addon/infrastructure/rpc_server.py`: threaded RPC server that schedules work safely on Blender's main thread.
- `tests/unit/`: fast tests with mocked Blender/RPC.
- `tests/e2e/`: Blender-backed end-to-end tests.
- `_docs/`: canonical design/task/history docs. Read the area-specific docs before structural changes.

## Architecture Rules

- Preserve Clean Architecture direction: `adapters -> application -> domain`.
- Do not import FastMCP, socket code, or Blender APIs into `server/domain/`.
- Keep Blender-specific logic inside `blender_addon/`.
- MCP adapters should stay thin. Business logic belongs in handlers or router components, not inside `@mcp.tool` wrappers.
- Dependency wiring belongs in `server/infrastructure/di.py`.
- Router additions must remain metadata-driven where possible.

## Runtime Boundaries

Read `_docs/_ROUTER/RESPONSIBILITY_BOUNDARIES.md` before changing FastMCP integration, LaBSE usage, router policy, or verification flows.

The intended responsibility split is:

- **FastMCP platform layer**: discovery, visibility, prompts, elicitation, background tasks, versioned/public MCP surfaces.
- **LaBSE semantic layer**: workflow matching, multilingual semantic retrieval, synonym handling, learned parameter reuse.
- **Router policy layer**: deterministic execution safety, mode/selection fixes, clamping, correction policy, ask/block/override decisions.
- **Inspection/assertion layer**: Blender truth and verification via scene/mesh/object inspection and future assertion tools.

Do not let these roles blur together:

- Do not use LaBSE as the authority for scene truth or execution safety.
- Do not use the router as the primary discovery/catalog-shaping mechanism when FastMCP platform features should handle that.
- Do not treat semantic confidence as proof that a Blender result is correct; rely on inspection/assertion tools for that.
- Prefer structured state reporting and verification over prose when correctness matters.

## Environment Notes

- Python `3.11+` is the practical baseline for full repo functionality.
- Router semantic features rely on `sentence-transformers`, `lancedb`, and `pyarrow`.
- LaBSE is heavy; shared DI instances and lazy initialization are intentional. Do not accidentally reintroduce per-test or per-call model loading.
- Blender `5.0` is the tested target. The addon declares Blender `4.0+`, but 4.x is best effort.

## Commands

- Install deps: `poetry install --no-interaction`
- Run server locally: `poetry run python server/main.py`
- Build addon zip: `python scripts/build_addon.py`
- Pre-commit install: `poetry run pre-commit install --hook-type pre-commit --hook-type pre-push`
- Full repo checks: `PRE_COMMIT_HOME=/tmp/pre-commit-cache poetry run pre-commit run --all-files` or `poetry run pre-commit run --all-files --show-diff-on-failure`
- Type checks: `poetry run mypy`
- Unit tests: `PYTHONPATH=. poetry run pytest tests/unit/ -v`
- Unit test collection count: `poetry run pytest tests/unit --collect-only`
- E2E tests: `python3 scripts/run_e2e_tests.py`
- E2E collection count: `poetry run pytest tests/e2e --collect-only`

## Coding Standards

- Fully type Python code.
- Tool docstrings are part of the product. Keep them explicit, accurate, and aligned with actual behavior.
- Prefer meaningful error strings over uncaught exceptions. The server should not crash on normal tool misuse.
- Follow naming patterns already used in the repo: `scene_*`, `modeling_*`, `mesh_*`, `router_*`, etc.
- Repo formatting/linting/type-checking is enforced through `pre-commit`. Keep `ruff`, `mypy`, JSON/TOML/YAML validation, GitHub workflow validation, and router metadata schema validation green.
- Do not weaken established domain/runtime contracts just to satisfy the type checker. If a contract is intentionally optional or dynamic, preserve it and narrow at call sites or use explicit helper functions/casts where needed.
- For RPC-backed handlers, prefer explicit result-unwrapping/narrowing helpers over changing `RpcResponse` semantics.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PatrykIti/blender-ai-mcp](https://github.com/PatrykIti/blender-ai-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
