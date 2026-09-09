---
trigger: always_on
description: Railtracks (`rt`) is a Python framework for building agentic systems: agents, tools, and multi-step flows
---

## Repository Overview

Railtracks (`rt`) is a Python framework for building agentic systems: agents, tools, and multi-step flows
defined entirely in standard Python (no YAML/DSLs). This is a `uv` workspace monorepo:

```
Root
├── packages/railtracks/          # The actual "railtracks" PyPI package
│   ├── pyproject.toml            # Package dependencies (add new deps here, in optional-dependencies for extras)
│   ├── src/railtracks/           # Source (module is railtracks, package dir uses underscore)
│   └── tests/                    # unit_tests/, integration_tests/, end_to_end/, llm_live_tests/
├── pyproject.toml                # Root workspace: dev-tooling deps only (docs/test/lint groups), NOT package deps
├── docs/                         # mkdocs documentation source
├── examples/                     # Example scripts
└── scripts/                      # CI helper scripts (dependency sorting, license checks, docs validation)
```

## Accessing docs

Full documentation lives at https://docs.railtracks.org/ (works even if this file is read standalone,
without the repo). Locally, the source is under `docs/documentation/` and `docs/observability/`; preview
with `mkdocs serve` (see Common Commands below). Doc URLs mirror the source path, e.g.
`docs/documentation/invocation/flows.md` -> `https://docs.railtracks.org/documentation/invocation/flows/`.

## Setup

```bash
uv sync --group dev
uv pip install -e "packages/railtracks[all]"   # or a specific extra, e.g. [visual], [retrieval]
```

## Common Commands

```bash
# Lint / format (must pass before commit, CI enforces this)
ruff check --fix
ruff format

# Unit tests only (fast, ~10s)
pytest packages/railtracks/tests/unit_tests/ -v --timeout=30

# Unit + integration tests (excludes llm_live_tests and end_to_end/retrieval, per root pyproject.toml addopts)
pytest -s -v packages/railtracks/tests/unit_tests/ packages/railtracks/tests/integration_tests/

# Single test file / test
pytest packages/railtracks/tests/unit_tests/nodes/test_x.py -v
pytest packages/railtracks/tests/unit_tests/nodes/test_x.py::test_name -v

# Dependency sort check (CI enforced)
python scripts/check_dependencies_sorted.py

# Docs
mkdocs serve            # local preview at localhost:8000
mkdocs build --strict --verbose
```

CI (`.github/workflows/pr_tests.yaml`) runs `ruff-lint` and `check-licenses` in parallel, plus a `changes`
job that path-filters which areas were touched. `unit_tests` (includes the integration tests and an inline
dependency-sort check), `retrieval_tests`, `documentation_validation` (`mkdocs build --strict`), and a
standalone `pyproject_dependency_order` job all wait on those three and only run when `changes` says their
area is affected. Run the lint/format/test commands above locally before pushing.

Note: `llm_live_tests` and `end_to_end/retrieval` require real API keys/network and are excluded from the
default pytest run via root `pyproject.toml` `addopts`; other `end_to_end` tests run by default.
`RAILTRACKS_TEST_MODE` is auto-enabled during tests (via `conftest.py`) to disable session persistence to
disk; opt into persistence testing with `RAILTRACKS_ALLOW_PERSISTENCE=1` and the `allow_persistence`
fixture.

## Architecture

### Core building blocks

For usage patterns (how to define tools/agents/flows, structured output, agent-as-tool, MCP tools), see
`packages/railtracks/src/railtracks/cli/skills/agent-builder.md` (the same content bundled for package
users via `railtracks add claude:agent-builder`) or the docs linked below; don't re-teach usage here. Just
where each concept lives internally, plus a doc link for the how-to:

- **Tool** (`rt.function_node`): `built_nodes/function/node.py`.
  [Function Tools](https://docs.railtracks.org/documentation/agent_design/tools/function_tools/).
- **Agent** (`rt.agent_node`): `built_nodes/llm/node.py`; always a single dynamically built node, no
  separate class per `tool_nodes`/`output_schema` combination. Passing both raises `NodeCreationError`.
  [Agent Design](https://docs.railtracks.org/documentation/agent_design/overview/).
- **Flow**: `orchestration/flow.py`.
  [Flows](https://docs.railtracks.org/documentation/invocation/flows/).
- **`rt.call(...)`**: `interaction/_call.py`, alongside `call_batch`/`astream`/`broadcast`/`couple`.
  [Call](https://docs.railtracks.org/documentation/invocation/call/).
- **Agent-as-tool**: `rt.ToolManifest(...)` in `nodes/manifest.py`.
  [Agents as Tools](https://docs.railtracks.org/documentation/agent_design/tools/agents_as_tools/).
- **Middleware**: `middleware/chain.py` (see "Things to avoid" below for the `middleware=`/
  `model_middleware=` gotcha). The LLM-call hooks were renamed `before_llm`/`after_llm` ->
  `pre_llm`/`post_llm` and `after_node` -> `post_node`; old names still work as deprecated shims.
  [Middleware](https://docs.railtracks.org/documentation/agent_design/middleware/overview/).
- **Human-in-the-loop verifier**: split into `rt.prebuilt.middleware.pre_verifier`/`post_verifier`
  (`prebuilt/middleware/pre_verifier.py`/`post_verifier.py`); `Verdict`/`VerifierRejectedError` still in
  `middleware/verdict.py`. There's no top-level `rt.verifier` anymore.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RailtownAI/railtracks](https://github.com/RailtownAI/railtracks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
