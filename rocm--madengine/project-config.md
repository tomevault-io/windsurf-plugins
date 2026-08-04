---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Behavioral Guidelines

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## Development Setup

```bash
# Install in development mode (all dependencies, including Kubernetes
# support and dev tools, are included by default)
pip install -e .

# Setup pre-commit hooks
pre-commit install
```

## Commands

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/unit/test_error_handling.py -v

# Run specific test class or function
pytest tests/unit/test_error_handling.py::TestErrorPatternMatching -v

# Run tests with coverage
pytest --cov=src/madengine --cov-report=html

# Skip slow tests
pytest -m "not slow"

# Format code
black src/ tests/
isort src/ tests/

# Lint
flake8 src/ tests/

# Type check
mypy src/madengine

# Run all pre-commit checks
pre-commit run --all-files
```

## Architecture

madengine is a CLI tool for running AI/ML models in local Docker, Kubernetes, and SLURM environments. The entry point is `madengine.cli.app:cli_main` (registered as the `madengine` console script).

### Layer Structure

**CLI Layer** (`src/madengine/cli/`)
- `app.py` — Typer app wiring, registers 5 commands: `discover`, `build`, `run`, `report`, `database`
- `commands/` — One file per command (build, run, discover, report, database)
- `constants.py` — `ExitCode` enum (`SUCCESS=0`, `FAILURE=1`, `BUILD_FAILURE=2`, `RUN_FAILURE=3`, `INVALID_ARGS=4`)

**Orchestration Layer** (`src/madengine/orchestration/`)
- `build_orchestrator.py` — `BuildOrchestrator`: discovers models, builds Docker images, writes `build_manifest.json`
- `run_orchestrator.py` — `RunOrchestrator`: reads or triggers builds, infers deployment target, delegates to local or distributed execution

**Core Layer** (`src/madengine/core/`)
- `context.py` — `Context` class: merges `additional_context` with system detection (GPU vendor, architecture, OS, ROCm path). Uses `ast.literal_eval()` to parse additional_context strings (not `json.loads` — pass Python dict repr, not JSON)
- `console.py` — `Console`: shell execution wrapper with live output support
- `docker.py` — Docker command wrapper

**Execution Layer** (`src/madengine/execution/`)
- `container_runner.py` — `ContainerRunner`: runs models from manifest via `docker run`, writes results to `perf.csv`
- `docker_builder.py` — `DockerBuilder`: builds images from Dockerfiles
- `container_runner_helpers.py` — Log error pattern scanning, timeout resolution

**Deployment Layer** (`src/madengine/deployment/`)
- `factory.py` — `DeploymentFactory`: Factory pattern, registers `SlurmDeployment` and `KubernetesDeployment`
- `base.py` — `BaseDeployment` abstract class, `DeploymentConfig` dataclass
- `kubernetes.py` / `slurm.py` — Concrete deployments; target is inferred by Convention over Configuration: presence of `"k8s"` or `"kubernetes"` key → K8s; `"slurm"` key → SLURM; neither → local

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ROCm/madengine](https://github.com/ROCm/madengine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
