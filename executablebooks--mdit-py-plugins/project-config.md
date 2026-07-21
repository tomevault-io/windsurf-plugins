---
trigger: always_on
description: This file provides guidance for AI coding agents working on the **mdit-py-plugins** repository.
---

# AGENTS.md

This file provides guidance for AI coding agents working on the **mdit-py-plugins** repository.

## Project Overview

mdit-py-plugins is a collection of plugins for [markdown-it-py](https://github.com/executablebooks/markdown-it-py), the Python Markdown parser. It provides:

- Syntax extensions for markdown-it-py (footnotes, front matter, definition lists, task lists, etc.)
- Both block-level and inline-level parsing rules
- Each plugin follows the markdown-it-py plugin architecture: a function that receives a `MarkdownIt` instance and registers rules
- Used extensively by [MyST-Parser](https://github.com/executablebooks/MyST-Parser) for Sphinx documentation

## Repository Structure

```
pyproject.toml          # Project configuration and dependencies (flit)
tox.ini                 # Tox test environment configuration
.pre-commit-config.yaml # Pre-commit hooks configuration

mdit_py_plugins/        # Main source code
├── __init__.py         # Package init (version only)
├── utils.py            # Shared utility functions
├── py.typed            # PEP 561 marker
├── admon/              # Admonition plugin (note, warning, etc.)
├── amsmath/            # AMS math environment plugin
├── anchors/            # Heading anchor plugin
├── attrs/              # Inline/block attribute plugin ({.class #id})
├── colon_fence.py      # Colon fence plugin (:::)
├── container/          # Container plugin (custom divs)
├── deflist/            # Definition list plugin
├── dollarmath/         # Dollar math plugin ($...$, $$...$$)
├── field_list/         # Field list plugin
├── footnote/           # Footnote plugin
├── front_matter/       # YAML front matter plugin
├── myst_blocks/        # MyST block syntax plugin
├── myst_role/          # MyST role syntax plugin ({role}`text`)
├── subscript/          # Subscript plugin (~sub~)
├── substitution.py     # Substitution plugin
├── tasklists/          # Task list plugin (- [x] done)
├── texmath/            # TeX math plugin
└── wordcount/          # Word count plugin

tests/                  # Test suite
├── fixtures/           # Shared test fixture files
├── test_admon/         # Admonition tests with fixture files
├── test_amsmath/       # AMS math tests
├── test_anchors.py     # Anchors tests
├── test_attrs/         # Attributes tests
├── test_colon_fence/   # Colon fence tests
├── test_container/     # Container tests
├── test_deflist/       # Definition list tests
├── test_dollarmath/    # Dollar math tests
├── test_field_list/    # Field list tests
├── test_footnote.py    # Footnote tests
├── test_front_matter.py # Front matter tests
├── test_myst_block.py  # MyST block tests
├── test_myst_role.py   # MyST role tests
├── test_subscript.py   # Subscript tests
├── test_substitution/  # Substitution tests
├── test_tasklists/     # Task list tests
├── test_texmath/       # TeX math tests
└── test_wordcount.py   # Word count tests

docs/                   # Documentation source
├── conf.py             # Sphinx configuration
└── index.md            # Documentation index
```

## Development Commands

All commands should be run via [`tox`](https://tox.wiki) for consistency. The project uses `tox-uv` for faster environment creation.

### Testing

```bash
# Run all tests
tox

# Run tests with specific Python version
tox -e py311

# Run a specific test file
tox -e py310 -- tests/test_footnote.py

# Run a specific test function
tox -e py310 -- tests/test_footnote.py::test_function_name

# Run tests with coverage
tox -e py310 -- --cov=mdit_py_plugins --cov-report=html

# Update regression test fixtures (produces error if files change)
tox -e py310 -- --force-regen
```

### Documentation

```bash
# Build docs (clean)
tox -e docs-clean

# Build docs (incremental)
tox -e docs-update

# Build with a specific builder (e.g., linkcheck to validate external links)
tox -e docs-update -- linkcheck
```

### Code Quality

```bash
# Run all pre-commit hooks (ruff, mypy, trailing whitespace, etc.)
tox -e pre-commit

# Run pre-commit with specific hook
tox -e pre-commit -- --all-files ruff

# Run mypy type checking
tox -e mypy

# Run pre-commit hooks directly (if pre-commit is installed)
pre-commit run --all-files
```

## Code Style Guidelines

- **Formatter/Linter**: Ruff (configured in `pyproject.toml`)
- **Type Checking**: Mypy with strict settings (configured in `pyproject.toml`)
- **Pre-commit**: Use pre-commit hooks for consistent code style

### Best Practices

- **Type annotations**: Use complete type annotations for all function signatures.
- **Docstrings**: Include docstrings for public functions and classes.
- **Pure functions**: Where possible, write pure functions without side effects.
- Follow existing naming conventions (`N802`, `N803`, `N806` are excluded to match markdown-it naming).

## Testing Guidelines

- **Framework**: pytest with `pytest-regressions` for fixture-based tests
- **Fixture pattern**: Most plugins use `.md` fixture files containing input and expected HTML output, separated by `.` markers
- **Test location**: Each plugin has a corresponding test file or directory in `tests/`
- **Regression tests**: Use `pytest-regressions` for comparing rendered output against stored fixtures

### Fixture File Format


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [executablebooks/mdit-py-plugins](https://github.com/executablebooks/mdit-py-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
