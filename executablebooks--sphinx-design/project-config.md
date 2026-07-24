---
trigger: always_on
description: This file provides guidance for AI coding agents working on the **sphinx-design** repository.
---

# AGENTS.md

This file provides guidance for AI coding agents working on the **sphinx-design** repository.

## Project Overview

sphinx-design is a Sphinx extension for designing beautiful, view size responsive web components. It provides:

- Grids, cards, dropdowns, tabs, badges, buttons, and icons
- Responsive design inspired by [Bootstrap](https://getbootstrap.com/) (v5), [Material Design](https://material.io), and [Material-UI](https://material-ui.com/)
- Support for both reStructuredText and [MyST Markdown](https://myst-parser.readthedocs.io/) (via integration with myst-parser)

The extension works with multiple Sphinx themes including alabaster, sphinx-rtd-theme, pydata-sphinx-theme, sphinx-book-theme, furo, and sphinx-immaterial.

## Repository Structure

```
pyproject.toml          # Project configuration and dependencies
tox.ini                 # Tox test environment configuration

sphinx_design/          # Main source code
├── __init__.py         # Package init with setup() entry point
├── _compat.py          # Compatibility utilities
├── extension.py        # Main Sphinx extension setup
├── shared.py           # Shared constants and base classes
├── badges_buttons.py   # Badge and button directives
├── cards.py            # Card directives
├── dropdown.py         # Dropdown directive
├── grids.py            # Grid directives
├── tabs.py             # Tab directives
├── icons.py            # Icon roles (Material, FontAwesome, Octicons)
├── article_info.py     # Article info directive
├── compiled/           # Bundled icon data (JSON)
└── static/             # Served static assets (compiled CSS, JS)

style/                  # CSS sources (compiled by tools/generate_css.py)
├── design.toml         # Declarative tokens for the generated utility families
├── cards.css           # Hand-authored card styles
├── tabs.css            # Hand-authored tab styles
├── dropdown.css        # Hand-authored dropdown styles
└── ...                 # Other hand-authored component styles

tools/                  # Dev-only tooling (not shipped in the wheel)
├── generate_css.py     # Builds sphinx_design/static/sphinx-design.min.css
└── check_css_equivalence.py  # CSS rule-set equivalence verifier (tinycss2)

tests/                  # Test suite
├── conftest.py         # Pytest fixtures
├── test_snippets.py    # Snippet-based tests
├── test_misc.py        # Miscellaneous tests
├── test_snippets/      # Test fixture files for snippets
└── test_misc/          # Test fixture files for misc tests

docs/                   # Documentation source (MyST Markdown)
├── conf.py             # Sphinx configuration
├── index.md            # Documentation index
├── get_started.md      # Getting started guide
├── grids.md            # Grid documentation
├── cards.md            # Card documentation
├── tabs.md             # Tab documentation
├── dropdowns.md        # Dropdown documentation
├── badges_buttons.md   # Badge and button documentation
└── snippets/           # Code snippets for docs (myst/, rst/)
```

## Development Commands

All commands should be run via [`tox`](https://tox.wiki) for consistency. The project uses `tox-uv` for faster environment creation.

### Testing

```bash
# Run all tests (default Python version)
tox

# Run tests with a specific Python version
tox -e py311
tox -e py312

# Run a specific test file
tox -- tests/test_snippets.py

# Run a specific test function
tox -- tests/test_snippets.py::test_function_name

# Run tests without myst-parser dependency
tox -e py311-no-myst

# Run with coverage
tox -- --cov=sphinx_design

# Update regression test fixtures
tox -- --force-regen
```

### Documentation

```bash
# Build docs with different themes
tox -e docs-alabaster
tox -e docs-rtd
tox -e docs-pydata
tox -e docs-sbt
tox -e docs-furo
tox -e docs-im

# Clean build (set CLEAN env var)
CLEAN=1 tox -e docs-furo

# Build with a different builder (e.g., linkcheck)
BUILDER=linkcheck tox -e docs-furo
```

### Code Quality

```bash
# Type checking with mypy
tox -e mypy

# Linting with ruff (auto-fix enabled)
tox -e ruff-check -- --fix

# Formatting with ruff
tox -e ruff-fmt

# Run pre-commit hooks on all files
pre-commit run --all-files

# Regenerate the compiled CSS artifact
python tools/generate_css.py
# or via pre-commit
pre-commit run --all-files css
```

## Code Style Guidelines

- **Formatter/Linter**: Ruff (configured in `pyproject.toml`)
- **Type Checking**: Mypy (configured in `pyproject.toml`)
- **Pre-commit**: Use pre-commit hooks for consistent code style

### Best Practices

- **Type annotations**: Use complete type annotations for all function signatures.
- **Docstrings**: Use Sphinx-style docstrings (`:param:`, `:return:`, `:raises:`).
- **Directive classes**: Extend `SdDirective` (from `shared.py`) for new directives.
- **Warning messages**: Use `WARNING_TYPE = "design"` for consistent warning categorization.
- **Testing**: Write tests for all new functionality. Use `pytest-regressions` for output comparison.

### Docstring Example

```python
def create_component(
    name: str,
    rawtext: str,
    *,
    classes: Sequence[str] = (),
) -> nodes.container:
    """Create a component container node.

    :param name: The component name.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [executablebooks/sphinx-design](https://github.com/executablebooks/sphinx-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
