---
trigger: always_on
description: **harness-evals** is an open-source AI evaluation framework for LLM agents, prompts, and structured outputs. It provides a `pip install`-able scoring engine with 70+ metrics across deterministic, structural, operational, reliability, predictability, MCP, similarity, LLM-judged, RAG, safety, agent, conversation, and security categories.
---

# AGENTS.md - harness-evals

## Project Overview

**harness-evals** is an open-source AI evaluation framework for LLM agents, prompts, and structured outputs. It provides a `pip install`-able scoring engine with 70+ metrics across deterministic, structural, operational, reliability, predictability, MCP, similarity, LLM-judged, RAG, safety, agent, conversation, and security categories.

**Core principle**: An eval always produces a `Score`. Every metric is a single class with a `measure()` method.

**Data flow**: `Golden` (authored) + agent output -> `EvalCase` (evaluated) -> `Score` (result)

**Language**: Python 3.10+
**License**: Apache 2.0
**Package name**: `harness-evals`
**Import name**: `harness_evals`

## Build System

```bash
pip install -e "."                # Core only (deterministic metrics, no LLM key needed)
pip install -e ".[llm]"           # + OpenAI, Anthropic for LLM-judged metrics
pip install -e ".[otlp]"          # + OTLP metrics & traces export
pip install -e ".[langfuse]"      # + Langfuse source/sink
pip install -e ".[similarity]"    # + BLEU metric (nltk)
pip install -e ".[harness]"       # + Harness AI Service LLM provider
pip install -e ".[benchmarks]"    # + Academic benchmarks (MMLU, GSM8K, HumanEval, etc.)
pip install -e ".[all]"           # Everything
pip install -e ".[all,dev]"       # Everything + dev tools
```

**Build tool**: Poetry via `pyproject.toml` (backend: `poetry-core`)
**No compiled extensions** — pure Python.

## Testing

```bash
pytest tests/ -v                              # All tests
pytest tests/ -v -m unit                      # Unit tests only
pytest tests/metrics/ -v                      # Specific directory
pytest tests/test_core.py -v                  # Specific file
pytest tests/test_core.py::test_evaluate -v   # Specific function
pytest tests/ --cov=harness_evals --cov-report=html  # With coverage
```

- Mark tests: `@pytest.mark.unit`, `@pytest.mark.integration`
- Test data: `tests/data/`

**ALWAYS run `pytest tests/ -v` before committing.**

## Linting & Formatting

```bash
ruff check src/ tests/           # Lint check
ruff format --check src/ tests/  # Format check
ruff format src/ tests/          # Auto-format
ruff check --fix src/ tests/     # Auto-fix lint issues
```

Ruff handles both formatting and linting (replaces black + flake8 + isort).

## Publishing

The package is published automatically via the Harness CI pipeline (`.harness/publish.yaml`) when a version change is detected on `main`.

**How it works**: The pipeline compares `version` in `pyproject.toml` at `HEAD` vs `HEAD~1`. If the version changed, it builds and publishes to `harness-pip-internal`.

**You MUST bump the version in `pyproject.toml`** whenever your changes should be released. If you don't bump the version, the package will NOT be published — even if code changes are merged.

```bash
# In pyproject.toml, update:
version = "X.Y.Z"  # Bump this to trigger a publish
```

Follow semver: patch for fixes, minor for new metrics/features, major for breaking changes.

## Git Workflow

- **Branch naming**: `feat/short-description` or `fix/short-description`
- **Commit format**: `type: description` where type is `feat`, `fix`, `chore`, `refactor`, `test`, `docs`
- **Default branch**: `main`

## DOs

- Use type hints on all function signatures
- Follow existing patterns — look at any metric file as a template
- Use `@dataclass` for structured data (Golden, EvalCase, Score)
- Keep metrics as single-file, single-class modules
- Write a test file for every new metric
- Use async/await for I/O (LLM calls, HTTP) — override `a_measure()` for async metrics
- Run `ruff check` and `pytest` before committing

## DON'Ts

- Never force push to main
- Never commit secrets or `.env` files
- Never add heavy dependencies (torch, transformers) to core — use optional extras
- Never modify `Golden`, `EvalCase`, or `Score` fields without updating PLAN.md
- Never average safety scores into an overall score — report them separately
- Don't use `print()` — use the sink system for output
- Never reference internal Harness repos, services, environments, or design docs

## Project Structure

```
harness-evals/
├── pyproject.toml                   # Package config (Poetry), dependencies, tool settings
├── README.md                        # User-facing documentation
├── AGENTS.md                        # This file
├── PLAN.md                          # Full vision spec
├── CHANGELOG.md                     # Version history
├── LICENSE                          # Apache 2.0
├── .harness/publish.yaml            # CI pipeline for publishing
│
├── src/harness_evals/
│   ├── __init__.py                  # Public API re-exports
│   ├── py.typed                     # PEP 561 marker
│   ├── cli.py                       # CLI entry point (harness-evals command)
│   ├── eval.py                      # run_eval() one-liner
│   ├── catalog.py                   # Metric catalog/registry

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harness/harness-evals](https://github.com/harness/harness-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
