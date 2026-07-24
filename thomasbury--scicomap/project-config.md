---
trigger: always_on
description: Operating guide for agentic coding tools in this repository.
---

# AGENTS.md
Operating guide for agentic coding tools in this repository.

## Project Overview
- Language: Python (`>=3.10`)
- Package root: `src/scicomap`
- Tests: `tests/`
- Docs source: `docs/source`
- Main tooling: `uv`
- Packaging config: `pyproject.toml`

## Instruction Priority
Apply guidance in this order:
1. Direct user instruction
2. This `AGENTS.md`
3. Existing repository patterns

## Cursor/Copilot Rule Files
Checked and currently absent:
- `.cursorrules`
- `.cursor/rules/`
- `.github/copilot-instructions.md`
If these files are added later, treat them as high-priority repo guidance.

## Setup Commands
Run from repository root.
```bash
uv lock
uv sync --extra lint --extra test --extra docs
```
Notes:
- Notebook docs via `nbsphinx` require `pandoc` installed.
- Use `uv run ...` for all project commands.

## Preferred Command Surface
Prefer `just` recipes when available (single source of truth):
```bash
just check
just docs
just marimo
just release-check
```

## Tooling Guardrails
- Use `uv`/`just` for project commands. Avoid ad-hoc environments.
- Use `gh` for GitHub operations (runs, releases, workflow status).
- Use `git` safely: no destructive operations unless user explicitly asks.
- Before pushing a stable release tag, verify publish workflow status with `gh run`.
- For framework-specific work (Marimo, Sphinx, Typer, Ruff, GitHub Actions, etc.), consult Context7 first for current guidance.

## Build, Lint, and Test Commands
### Standard local checks
```bash
uv run python -m pytest
uv run ruff check src tests
uv run ruff format --check src tests
```

### Single-test workflows (important)
```bash
# single test file
uv run python -m pytest tests/docs/test_build_llm_assets.py

# single test function
uv run python -m pytest tests/docs/test_build_llm_assets.py::test_to_markdown_does_not_duplicate_h1

# run tests by keyword expression
uv run python -m pytest -k "sidebar_skip"
```

### Docs and LLM asset pipeline
```bash
uv run sphinx-build -n -b html docs/source docs/build/html
uv run python scripts/build_llm_assets.py
uv run python -m pytest tests/docs/test_build_llm_assets.py
```

### CI-style docs asset validation
```bash
uv run python -c "from pathlib import Path; root=Path('docs/build/html'); llms=root/'llms.txt'; md=list((root/'llm').rglob('*.md')); assert llms.exists(), 'llms.txt missing'; assert md, 'No markdown mirrors generated'"
```

### Optional Makefile path
```bash
make -C docs html
```

## Codebase Landmarks
- `src/scicomap/scicomap.py`: public classes and plotting entry points
- `src/scicomap/cmath.py`: color math and colormap transforms
- `src/scicomap/cblind.py`: color-vision deficiency transforms
- `src/scicomap/utils.py`: plot/data helper functions
- `src/scicomap/datasets.py`: packaged data loaders
- `scripts/build_llm_assets.py`: docs HTML to markdown mirror + `llms.txt`
- `tests/docs/test_build_llm_assets.py`: parser regression tests

## Marimo/WASM Guardrails
- Never access `ui_element.value` in the same cell where that UI element is created.
- Browser WASM runtime is isolated from the `uv` environment.
- In `docs/marimo/tutorial_app_lite.py`, install runtime deps inside notebook cells (for example, via `micropip`).
- Enforce install-before-import with explicit cell dependency edges (sentinel variable).
- Do not rely on local helper-module imports for exported WASM unless runtime packaging/path is explicitly verified.

## Code Style Guidelines
Follow existing style in touched files. For new code, use these defaults.

### Imports
- Prefer absolute package imports (e.g., `from scicomap.cmath import ...`).
- Group imports by standard library, third-party, local package.
- Avoid wildcard imports in new code.
- Keep import lists tidy and remove unused names.

### Formatting
- Use `ruff format` and keep `ruff check` clean.
- Use 4-space indentation.
- Prefer readable line breaks over dense one-liners.
- Keep function bodies focused; extract helpers when logic grows.

### Types
- Add type hints to new/modified public APIs.
- Keep annotations consistent inside a file.
- Use precise return types, including tuple shapes where useful.
- Prefer explicit optionality (`X | None` or `Optional[X]`) consistently.

### Naming
- Classes: `CamelCase`
- Functions/variables: `snake_case`
- Constants: `UPPER_SNAKE_CASE`
- Internal helpers: leading underscore allowed and common here.
- Preserve established domain names (`cmap`, `Jpapbp`, `uniformized`).

### Docstrings
- Use NumPy-style docstrings for new/modified public modules, classes, functions, and methods.
- Preferred section order: summary, optional extended summary, `Parameters`, `Returns`/`Yields`, `Raises` (when applicable), optional `Notes`, optional `Examples`.
- Keep parameter names and defaults synchronized with function signatures and implementation behavior.
- In `Parameters`, include concrete constraints when non-obvious (accepted values, shapes, units, range).
- In `Returns`/`Yields`, describe output type and shape for arrays/tuples when relevant.
- In `Raises`, document specific exception types and the condition that triggers them.
- Include short examples only when non-obvious behavior needs clarification.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ThomasBury/scicomap](https://github.com/ThomasBury/scicomap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
