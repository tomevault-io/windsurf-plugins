---
trigger: always_on
description: MASEval is an orchestration library for benchmarking LLM-based multi-agent systems. Think of it as PyTorch Lightning for agent evaluation—it provides the execution engine while users implement agent logic.
---

# AGENTS.md

## Project Overview

MASEval is an orchestration library for benchmarking LLM-based multi-agent systems. Think of it as PyTorch Lightning for agent evaluation—it provides the execution engine while users implement agent logic.

**Key Architecture Rule:** Strict separation between `maseval/core` (minimal dependencies) and `maseval/interface` (optional integrations). Core must NEVER import from interface.

The library is in early development, so breaking changes that are parsimonous are strongly preferred.

## Setup Commands

```bash
# Sync environment with all dependencies (creates .venv automatically)
uv sync --all-extras --all-groups

# Activate environment
source .venv/bin/activate

# Or use uv run for individual commands (no activation needed)
uv run python examples/amazon_collab.py
uv run pytest tests/
```

## Code Style and Quality

- Line length: 144 characters
- Tool: `ruff`
- All checks must pass in CI before merge

```bash
# Format code
uv run ruff format .

# Lint and auto-fix issues
uv run ruff check . --fix
```

## Testing Instructions

- Tests use composable pytest markers — see `tests/README.md` for full details
- **What it tests**: `core`, `interface`, `contract`, `benchmark`, `smolagents`, `langgraph`, `llamaindex`, `gaia2`, `camel`
- **What it needs**: `live` (network), `credentialed` (API keys), `slow` (>30s), `smoke` (full pipeline)
- Default `pytest` excludes `slow`, `credentialed`, and `smoke` via `addopts`
- All tests must pass before PR merge
- Add/update tests for code changes
- **Benchmark tests** follow a two-tier pattern (offline structural + live real-data). See `tests/README.md` for the recommended pattern when adding or modifying benchmark tests.

```bash
# Default — fast tests only
uv run pytest -v

# Core tests only (minimal dependencies)
uv run pytest -m core -v

# Specific integration tests
uv run pytest -m smolagents -v
uv run pytest -m interface -v

# Data download validation (needs network)
uv run pytest -m "live and slow" -v

# Live API tests (needs OPENAI_API_KEY, ANTHROPIC_API_KEY, GOOGLE_API_KEY)
uv run pytest -m credentialed -v

# Fully offline
uv run pytest -m "not live" -v
```

## Coverage

View coverage by feature area (auto-discovers benchmarks/interfaces):

```bash
# Full coverage (default + slow + live, excludes credentialed and smoke)
uv run python scripts/coverage_by_feature.py

# Fast-only (skip slow and live tests)
uv run python scripts/coverage_by_feature.py --exclude slow,live
```

Manual coverage for specific modules:

```bash
pytest --cov=maseval.core.agent --cov-report=term-missing
```

## Typing

### Type Checker

The project uses the `ty` type checker.

```bash
# Check types across the project
uv run ty check

# View documentation
uv run ty --help
```

Documentation: [https://docs.astral.sh/ty/](https://docs.astral.sh/ty/)

### Philosophy & Priorities

**Types exist to help users and catch bugs—not to satisfy theoretical purity.**

This library uses type hinting to:

- Provide better IDE autocomplete and error detection
- Document expected types clearly
- Catch real errors before runtime

However, **pragmatism over pedantry**: if a typing pattern improves usability and robustness, use it—even if it technically violates some typing rule.

**Example:** `MACSBenchmark` narrows its environment type to `MACSEnvironment` (instead of generic `Environment`). This violates strict subtyping rules but provides users with:

- Precise autocomplete for MACS-specific methods
- Clear documentation that MACS requires its own environment
- Better error messages during development
- Prevents mixing incompatible components (e.g., `Tau2Environment` cannot be passed to `MACSBenchmark`)

Unless there's a graceful alternative that preserves usability, choose the pattern that helps users most.

**Guiding principle:** Orient yourself on existing patterns in the codebase. Consistency matters more than theoretical correctness.

### Syntax Rules

- **Unions:** Use `A | B` notation (not `Union[A, B]`)
- **Optional:** Prefer `Optional[X]` over `X | None` for explicitness
- **Collections:** Use `List[...]`, `Dict[..., ...]`, `Sequence[...]` instead of `list`, `dict`, `sequence`

**Example:**

```python
def process_data(
    items: List[str],
    config: Optional[Dict[str, Any]] = None
) -> str | int:
    ...
```

## Dependency Management

Three types of dependencies:

- **Core** (`[project.dependencies]`): Required, installed with `pip install maseval`. Keep minimal!
- **Optional** (`[project.optional-dependencies]`): Published for end users. Framework integrations like `smolagents`, `langgraph`, plus convenience bundles like `all` and `examples`. Uses self-references (e.g., `maseval[all]`) to avoid duplication - this is a standard Python packaging feature.
- **Dev Groups** (`[dependency-groups]`): NOT published. Only for contributors. Tools like `pytest`, `ruff`, `mkdocs`.

```bash
# Add core dependency (use sparingly!)
uv add <package-name>

# Add optional dependency for end users (e.g., framework integrations)
uv add --optional <extra-name> <package-name>


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parameterlab/MASEval](https://github.com/parameterlab/MASEval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
