---
trigger: always_on
description: handles it.
---

# AGENTS.md

Role-based guidance for AI agents working in this repository.

PhysioMotion4D converts 4D CT scans into animated USD models for NVIDIA
Omniverse. It is an **early-alpha** scientific Python library. Clarity beats
premature optimization. Breaking changes are acceptable. Backward compatibility
is not a goal.

## Role

We are developing open-source code for scientific AI libraries. Leverage
GPU-accelerated methods when appropriate.

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

Prefer the repository-local virtual environment at `.\venv`. Activate it before
issuing Python commands so `python`, console scripts, and `uv pip` all use that
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
python -m pytest tests/ -v --run-experiments
python -m pytest tests/ -v --run-tutorials

# Enable every bucket at once (equivalent to passing all --run-* flags)
python -m pytest tests/ -v --run-all

# Typical local GPU profile
python -m pytest tests/ -v --run-gpu --run-slow

# Coverage
python -m pytest tests/ --cov=src/physiomotion4d --cov-report=html

# Create missing baselines
python -m pytest tests/ --create-baselines
```

Version bumping: `bumpver update --patch`, `--minor`, or `--major`.

## Codex Sandbox

- If a Python command fails with
  `No Python at '"C:\Users\saylward\AppData\Local\Programs\Python\Python311\python.exe'`,
  do not assume Python 3.11 is missing. The Codex sandbox can break the
  launcher or venv path.
- Use the temporary validation environment instead:
  `C:\Users\saylward\AppData\Local\Temp\pm4d-uv-env\Scripts\python.exe`.
  It has been verified as Python 3.11.9 with `ruff` available.
- Run that temporary venv outside the sandbox when needed. Treat this as an
  environment/sandbox workaround, not a dependency or installation problem.

## Universal Rules

- Read the relevant source files before proposing changes.
- Runtime classes for workflows, segmentation, registration, and USD tools
  inherit from `PhysioMotion4DBase`; new runtime classes must too. Standalone
  utility scripts and data/container/helper classes do not.
- In classes that inherit from `PhysioMotion4DBase`, use `self.log_info()` and
  `self.log_debug()`, never `print()`. Standalone scripts may use `print()`.
- No emojis in `.py` files; avoid them in docs too. Windows cp1252 encoding
  has broken this project before.
- The public VTK→USD entry point is `ConvertVTKToUSD`. Experiments, CLIs,
  tests, and tutorials must use it. Do not import from the `vtk_to_usd/`
  subpackage directly outside of `convert_vtk_to_usd.py` and the subpackage
  itself.
- Scripts that instantiate `SegmentChestTotalSegmentator` must guard the
  top-level invocation with `if __name__ == "__main__":` on Windows
  (`torch.multiprocessing` requires it).
- Double quotes for strings and docstrings. Keep lines at or
  below 88 characters.
- Full type hints are required under strict mypy. Use `Optional[X]`, not
  `X | None`.
- Run `python -m pytest tests/ -v` from the active `.\venv` to verify changes.
  Slow, GPU, Simpleware,
  experiment, and tutorial tests are auto-skipped unless their opt-in flag is
  passed.
- The `requires_data` marker no longer exists. Tests that need external data
  download it automatically via the session fixtures.
- Consult `docs/API_MAP.md` to locate classes, methods, and signatures before
  searching manually.
- Do not commit changes or make pull requests unless specifically told to do so.

## Data Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Project-MONAI/physiomotion4d](https://github.com/Project-MONAI/physiomotion4d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
