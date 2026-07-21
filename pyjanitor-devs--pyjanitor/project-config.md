---
trigger: always_on
description: This file provides guidance to LLM agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to LLM agents working with code in this repository.
It serves as the agent's "constitution" for pyjanitor development.

---

## Agent Constitution

### Self-Improvement Protocol

**CRITICAL RULE**: This file is a living document. Agents MUST update it when:

1. **User Corrections**: If the user corrects you on anything, immediately record
   the correction in this file (AGENTS.md) in an appropriate section, then
   continue with what you were doing, applying the correction.

2. **Discovered Patterns**: If you discover a pattern, convention, or best
   practice not documented here while working on the codebase, add it to the
   appropriate section.

3. **Command Updates**: If you find that a command has changed, been deprecated,
   or a better alternative exists, update the Commands section.

4. **Anti-Patterns**: If you make a mistake and learn from it, document the
   anti-pattern in the appropriate section to prevent future occurrences.

**How to Update**: Add new learnings to the `## Learned Patterns` section at
the bottom of this file. The maintainer will periodically review and integrate
these into the main sections.

### Core Principles

- **Read Before Edit**: Always read and understand relevant files before
  proposing changes.
- **Minimal Changes**: Make the smallest change necessary to accomplish the
  task.
- **Test-Driven**: Always run tests after making code changes.
- **Document**: Keep docstrings up-to-date using Google-style format.
- **Lint Markdown**: Always run `markdownlint` on markdown files after editing.

---

## Project Overview

pyjanitor is a Python implementation of the R package janitor. It provides a
clean, chainable API for extending pandas with powerful and readable
data-cleaning functions.

**Key Design Philosophy**:

- Methods are chainable (fluent interface)
- Methods are registered via `pandas_flavor` as DataFrame methods
- All methods return a DataFrame (immutability pattern - no mutation)
- Functions follow a consistent signature pattern: `df` first, then parameters

---

## Development Environment

### Package Manager

**This project uses `pixi` for dependency management and environment setup.**

**⚠️ CRITICAL FOR LLM AGENTS**: All Python commands MUST be run within a pixi
context. Never run Python commands directly without the `pixi run` prefix.

```bash
# ✅ CORRECT
pixi run python -c "import janitor; print(janitor.__version__)"
pixi run pytest tests/functions/test_clean_names.py -v
pixi run python scripts/my_script.py

# ❌ INCORRECT - will fail or use wrong environment
python -c "import janitor; ..."
pytest tests/
python scripts/my_script.py
```

### Environment Setup

```bash
# Enter development shell
pixi shell

# Or prefix individual commands
pixi run <command>
```

### Available Pixi Environments

| Environment | Purpose | Features |
|-------------|---------|----------|
| `default` | Standard development | tests, setup |
| `docs` | Documentation building | mkdocs, mkdocstrings |
| `tests` | Running test suite | pytest, hypothesis |
| `biology` | Biology module development | biopython |
| `chemistry` | Chemistry module development | rdkit, tqdm |
| `engineering` | Engineering module development | unyt |
| `spark` | PySpark development | pyspark |
| `py311`/`py312`/`py313` | Python version testing | Specific Python versions |

To run commands in a specific environment:

```bash
pixi run -e <environment> <command>
```

---

## Commands Reference

### Essential Commands

| Task | Command |
|------|---------|
| Run all tests | `pixi run test` |
| Run specific test | `pixi run pytest tests/functions/test_clean_names.py` |
| Run tests matching pattern | `pixi run pytest -k "test_clean_names" -v` |
| Run tests with coverage | `pixi run pytest --cov=janitor` |
| Build documentation | `pixi run docs` |
| Serve docs locally | `pixi run serve-docs` |
| Run linting | `pixi run lint` |
| Format code | `pixi run format` |
| Run all checks | `pixi run check` |
| Install pre-commit hooks | `pixi run start` |

### Testing Commands

```bash
# Run full test suite with parallel execution
pixi run pytest -v -n auto --color=yes

# Run tests for a specific module
pixi run pytest tests/functions/ -v
pixi run pytest tests/polars/ -v
pixi run pytest tests/chemistry/ -v

# Run doctests in source code
pixi run pytest --doctest-modules janitor/

# Run tests with specific marker
pixi run pytest -m "functions" -v
pixi run pytest -m "biology" -v
pixi run pytest -m "chemistry" -v

# Run a single test function
pixi run pytest tests/functions/test_clean_names.py::test_clean_names_method_chain
```

### Documentation Commands

```bash
# Build docs
pixi run docs

# Serve docs with live reload
pixi run serve-docs

# Build docs in specific environment
pixi run -e docs build-docs
```

### Code Quality Commands

```bash
# Run all pre-commit hooks
pixi run lint

# Format code with ruff
pixi run format

# Check import sorting
pixi run isort

# Run full style check
pixi run style
```

### Markdown Linting

**Always run `markdownlint` on markdown files after editing them.**

```bash
# Lint a markdown file
markdownlint AGENTS.md

# Lint all markdown files
markdownlint "**/*.md"

# If markdownlint is not on PATH, install it globally:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pyjanitor-devs/pyjanitor](https://github.com/pyjanitor-devs/pyjanitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
