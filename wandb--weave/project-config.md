---
trigger: always_on
description: - When you learn something new about the codebase or introduce a new concept, update this file (`AGENTS.md`) to reflect the new knowledge. This is YOUR FILE! It should grow and evolve with you.
---

# Agent instructions for `weave` repository

## Core Rules

- When you learn something new about the codebase or introduce a new concept, update this file (`AGENTS.md`) to reflect the new knowledge. This is YOUR FILE! It should grow and evolve with you.
- If there is something that doesn't make sense architecturally, devex-wise, or product-wise, please update the `Requests to Humans` section below.
- Always follow the established coding patterns and conventions in the codebase.
- Document any significant architectural decisions or changes.

## Python Import Rules

**IMPORTANT: Always place imports at the top of Python files.**

- All imports must be at the module level (top of the file), not inside functions or methods.
- The only exceptions are:
  - Circular import avoidance (must be documented with a comment explaining why)
  - Optional dependencies that may not be installed (must be wrapped in try/except)
  - TYPE_CHECKING imports for type hints only
- Note: Imports inside functions are not caught by linting. **Agents must self-enforce this rule.**

❌ **Never do this:**

```python
def my_function():
    import re  # BAD: import inside function
    return re.match(...)
```

✅ **Always do this:**

```python
import re  # GOOD: import at top of file

def my_function():
    return re.match(...)
```

## Development Setup

### Local Development (uv)

This project uses `uv` for dependency management. Dependencies are organized into **dependency-groups** (not extras) in `pyproject.toml`.

**Quick reference:**

- **Run tests**: `uv run --group test python -m pytest <path> -v`
- **Run linting**: `uvx ruff check <path>`
- **Run lint with auto-fix**: `uvx ruff check --fix <path>`

**Common pitfalls:**

- Do NOT use bare `python -m pytest` — the `python` on PATH may be from a uv cache, not the project `.venv`. Always use `uv run`.
- Do NOT use `--extra test` — `test` is a dependency-group, not an optional-dependency. Use `--group test`.
- `ruff` is not installed in any project dependency group. Use `uvx ruff` to run it.
- Ruff now enforces `PLW` rules. `PLW0602`, `PLW0603`, `PLW1641`, and `PLW3201` are handled with spot-level inline `# noqa` on specific lines (not global/per-file ignore). Prefer fixing code first; if intentional, suppress only the exact line.
- Be careful with `PLW1514` autofixes on serialization-sensitive code (`weave/type_handlers/Content/content.py`, `weave/type_handlers/Audio/audio.py`) and mocked file I/O (`weave/trace_server/costs/update_costs.py`): adding `encoding=` changed behavior/tests, so these files are explicitly ignored for that rule.

### Codex Development (nox)

- Your machine should be setup for you automatically via `bin/codex_setup.sh`
- If you encounter any setup issues:
  1. Check the setup script for potential problems
  2. Update `bin/codex_setup.sh` with necessary fixes
  3. Document any manual steps required in this section

_Important:_ For OpenAI Codex agents (most likely you!), your environment does not have internet access. If you need something setup beforehand, this is where you need to do it.

## Codebase Structure

### Main Components

- `weave/` - Core implementation
  - `weave/` - Python package implementation
  - `weave/trace_server` - Backend server implementation

## Generated Files — Do Not Hand-Edit

`weave/trace_server/model_providers/model_providers.json` and `weave/trace_server/costs/cost_checkpoint.json` are generated. Never edit them by hand — regenerate with `make update_model_providers` / `make update_costs` (see `weave/Makefile`).

Note: the scripts read `modelsBegin.json`/`modelsFinal.json`, which are symlinks into wandb/core and only resolve when this repo is checked out as the submodule inside wandb/core (`services/weave-trace/weave-python/weave-public`).

### Trace Server API / Node SDK Schema

When trace-server request/response models or route schemas change, refresh the API schema used by the Node SDK:

1. From this repo, run `make -C ../../weave-trace export-api-schema` to regenerate the sibling trace service's `openapi.json` from the FastAPI app.
2. Copy that schema into the tracked Node SDK schema: `cp ../../weave-trace/openapi.json sdks/node/weave.openapi.json`.
3. Regenerate the TypeScript client from `sdks/node`: `pnpm run generate-api`.

Evaluation result rows merge agent span links from two sources: legacy
`weave.genai_span_ref` call attributes and OTel spans whose promoted
`eval_run_id` plus `eval_predict_and_score_call_id` columns identify the
trial. Keep the promoted-column hydration best-effort so eval results remain
available during rolling deploys.

If `sdks/node/node_modules` is missing, run `pnpm install --frozen-lockfile` in `sdks/node` first. Do not use `npm install`; this SDK is pinned to pnpm.

## Python Testing Guidelines

### Test Framework

- Testing is managed by `nox` with multiple shards for different Python versions
- Each shard represents specific package configurations

### Server fixtures (do NOT hand-roll fake servers)

- **Never create a `_FakeServer`, stub, or mock `TraceServerInterface`** to test
  server-side logic. Use the existing `client` fixture (gives `client.server` +

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wandb/weave](https://github.com/wandb/weave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
