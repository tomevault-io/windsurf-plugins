---
trigger: always_on
description: handles it.
---

# AGENTS.md

Role-based guidance for AI agents working in this repository.

MONAI Physio is a collection of methods, workflows, tutorials, and CLI tools
for creating personalized physiological digital twins: starting from a 3D
medical image of a subject, extracting anatomic models, and then using AI
surrogates to estimate the subject's physiological processes (initially
cardiac and respiratory motion, expanding to electrophysiology, blood flow,
and organ perfusion). It is an **early-alpha** scientific Python library.
Clarity beats premature optimization. Prefer compatibility: break a public API
only when the change is generally beneficial to future users, and record every
break in the migration guide (see below).

## Role

We are developing open-source code for scientific AI libraries.

**A GPU is assumed.** Supporting CPU-only machines is not a requirement.
Design for the GPU first and use it wherever it is faster - do not add CPU
fallbacks, dtype compromises, or size limits to keep a CPU-only path viable,
and do not weaken an algorithm because a CPU could not run it.

It follows that tests and tutorials may require a GPU. Mark those that do with
`@pytest.mark.requires_gpu` so the bucket stays honest, but do not contort a
test to avoid the marker.

## Priorities

1. Accuracy.
2. Clarity, maintainability, and simplicity.
3. Consistency with the rest of the platform and open-source standards.
4. Documentation.
5. Testing.

## Behavior

1. Do not assume. Do not hide confusion. Surface tradeoffs.
2. Minimum code that solves the problem. Nothing speculative.
3. Touch only what you must. Clean up only your own mess.
4. Define success criteria. Loop until verified.

## Developer Tool Prerequisites

Non-Python tools used by contributor workflows:

- **Codex CLI** (`codex`) - can run the `.agents/` slash skills and is the
  default PR-review agent for `ai_agent_github_reviews.py`.
- **Claude Code CLI** (`claude`) - can run the `.agents/` slash skills and
  `ai_agent_github_reviews.py --agent claude`.
  Install: `winget install Anthropic.ClaudeCode`.
- **gh CLI** (`gh`) - required by `ai_agent_github_reviews.py` to fetch PR
  review data. Install: `winget install GitHub.cli` then `gh auth login`.
  Not installable via pip/uv; it is a compiled Go binary.

## Common Commands

Prefer the repository-local virtual environment at `.\venv` or `..\venv`. Activate it
before issuing Python commands so `python`, console scripts, and `uv pip` all use that
environment. If activation is not possible, invoke
`.\venv\Scripts\python.exe -m ...` directly. Use `uv run ...` only when the
local `venv` is unavailable and you need uv to create or sync an environment.

```powershell
# Create the repo-local environment if it does not already exist
uv venv venv
.\venv\Scripts\Activate.ps1

# Install in editable mode
uv pip install -e .

# Lint and format
python -m ruff check . --fix && python -m ruff format .

# Type checking
python -m mypy src/ tests/

# All pre-commit hooks
python -m pre_commit run --all-files

# Fast tests
python -m pytest tests/ -v

# Single test file or test by name
python -m pytest tests/test_contour_tools.py -v
python -m pytest tests/test_contour_tools.py::test_extract_surface -v

# Opt-in test buckets
python -m pytest tests/ -v --run-slow
python -m pytest tests/ -v --run-gpu
python -m pytest tests/ -v --run-simpleware
python -m pytest tests/ -v --run-physicsnemo
python -m pytest tests/ -v --run-tutorials

# Enable every bucket at once (equivalent to passing all --run-* flags)
python -m pytest tests/ -v --run-all

# Typical local GPU profile
python -m pytest tests/ -v --run-gpu --run-slow

# Coverage
python -m pytest tests/ --cov=src/monai_physio --cov-report=html

# Create missing baselines
python -m pytest tests/ --create-baselines
```

Version bumping: `bumpver update --patch`, `--minor`, or `--major`.

## Migration Guide

MONAI Physio prefers compatibility. Break a public API only when the change is
generally beneficial to future users. Never add deprecation shims,
removed-symbol re-exports, or removed-symbol stubs; when a break is
substantial, ship code that automates the conversion instead.

At commit time: if the diff breaks a public API, append an entry to
`docs/developer/migration_next.md` in that same commit - what changed, why it
benefits future users, before/after code, and the conversion script (or
`None needed`). Follow the entry template at the bottom of that file.

At release time:

```bash
bumpver update --patch
git mv docs/developer/migration_next.md docs/developer/migration_<new_version>.md
# retitle the archived file to "Migration Guide - <new_version>"
# recreate docs/developer/migration_next.md from its entry template
```

The `Developer Guides` toctree in `docs/index.rst` globs
`developer/migration_*`, so archived guides need no further wiring.

## graphify

This project keeps a knowledge graph at `graphify-out/` covering god nodes,
community structure, and cross-file relationships. It is the recommended way
to navigate the codebase with an AI assistant: a scoped subgraph is far
smaller and more accurate than raw grep output over 8,000+ lines of source.

```bash
graphify query "<question>"     # codebase questions -> scoped subgraph
graphify path "<A>" "<B>"       # how two symbols relate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Project-MONAI/monai-physio](https://github.com/Project-MONAI/monai-physio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
