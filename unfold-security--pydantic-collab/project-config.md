---
trigger: always_on
description: Core runtime code lives in `pydantic_collab/`. `collab.py` holds the orchestration engine, `_types.py` defines settings and runtime state, `_utils.py` contains prompt/context helpers, and `custom_collabs.py` provides ready-made topologies (forward chains, stars, meshes). Public docs live in `README.md` and `docs/topology_example.png`. `examples/` contains numbered Python scripts that demonstrate progressively more complex agent graphs and reuse the helper utilities in `examples/example_tools.py`
---

# Repository Guidelines

## Project Structure & Module Organization
Core runtime code lives in `pydantic_collab/`. `collab.py` holds the orchestration engine, `_types.py` defines settings and runtime state, `_utils.py` contains prompt/context helpers, and `custom_collabs.py` provides ready-made topologies (forward chains, stars, meshes). Public docs live in `README.md` and `docs/topology_example.png`. `examples/` contains numbered Python scripts that demonstrate progressively more complex agent graphs and reuse the helper utilities in `examples/example_tools.py`. Automated checks sit under `tests/` and mirror real collaboration scenarios (handoff control, topology validation, prompt output, import safety). Generated builds go to `dist/`, and `.logfire/` plus `.ruff_cache/` store tooling artifacts.

## Build, Test, and Development Commands

```bash
# Install & lock dependencies using uv.lock
uv sync

# Run the full pytest suite (includes asyncio-based scenarios)
uv run pytest

# Execute an example topology with the configured environment variables
uv run --env-file .env examples/01_simple_chain.py
```

## Coding Style & Naming Conventions
- **Indentation**: 4 spaces everywhere; tabs are not used.
- **File naming**: Modules follow `snake_case` (e.g., `_utils.py`, `custom_collabs.py`); tests use `test_*.py`.
- **Function/variable naming**: Snake case for functions (`default_build_agent_prompt`), PascalCase only for classes like `CollabAgent`.
- **Linting**: Ruff enforces a 120-character limit, Google-style docstrings, single quotes, import sorting, and extra rules (`Q`, `UP`, `I`, `D`, etc.). Run `uv run ruff check .` and `uv run ruff format .` before opening a PR. Pyright strict mode guards types: `uv run pyright`.

## Testing Guidelines
- **Framework**: Pytest with `pytest-asyncio` for async agents; tests occasionally use `pydantic_ai.models.test.TestModel` to stub LLMs.
- **Test files**: Located in `tests/`, following `test_<feature>.py` and using shared fixtures from `tests/fixtures_handoff.py`.
- **Running tests**: `uv run pytest` (ensure `.env` is present when tests depend on API keys).
- **Coverage**: No explicit threshold in repo—focus on exercising handoff paths, topology validation, and prompt builders before submissions.

## Commit & Pull Request Guidelines
- **Commit format**: History is minimal (`First Commit`), so follow conventional, imperative summaries such as `Add topology validation tests` or `Fix Collab handoff recursion`. Keep scope tight; run lint/type/test checks before committing.
- **PR process**: Include what topology or subsystem you touched, proof of local test runs, and screenshots/text for visualization changes. Reference relevant example scripts when adding new behaviors.
- **Branch naming**: Not predefined—use descriptive branches like `feature/handoff-context` or `fix/prompt-builder` so reviewers can infer scope quickly.

---

# Repository Tour

## 🎯 What This Repository Does

`pydantic-collab` is a declarative multi-agent orchestration framework for [pydantic-ai](https://ai.pydantic.dev/). It lets you design agent topologies (chains, stars, meshes, or fully custom graphs), govern tool-call vs. handoff behavior, and observe execution history with rich context.

**Key responsibilities:**
- Normalize arbitrary agent declarations into a validated collaboration graph
- Execute agents sequentially or via tool calls while enforcing limits (handoffs, call depth, usage)
- Provide ergonomic APIs for prompt/context customization, visualization, and testing

---

## 🏗️ Architecture Overview

### System Context
```
Client code (examples/tests) → Collab orchestration layer → pydantic-ai agents
                                       ↓
                                Optional observers (logfire, visualizers)
```

### Key Components
- **`Collab` (pydantic_collab/collab.py)** – Central orchestrator; validates graphs, manages handoffs/tool calls, tracks execution history, and exposes sync/async run APIs.
- **`CollabAgent` & settings (pydantic_collab/_types.py)** – Strongly-typed wrappers describing each agent’s capabilities, limits, and state; includes `CollabSettings`, `HandOffBase`, `CollabRunResult`, and runtime `CollabState`.
- **Prompt/context utilities (pydantic_collab/_utils.py)** – Convert message histories to text, extract agent-to-agent tool calls, and build descriptive prompts via `default_build_agent_prompt`.
- **Specialized topologies (pydantic_collab/custom_collabs.py)** – `PipelineCollab`, `StarCollab`, `MeshCollab`, and `HierarchyCollab` preconfigure `_build_topology` to reduce boilerplate.

### Data Flow
1. User declares `Collab` (or a custom subclass) with agent tuples or `CollabAgent` objects.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Unfold-Security/pydantic-collab](https://github.com/Unfold-Security/pydantic-collab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
