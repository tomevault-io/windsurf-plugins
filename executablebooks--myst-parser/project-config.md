---
trigger: always_on
description: This file provides guidance for AI coding agents working on the **MyST-Parser** repository.
---

# AGENTS.md

This file provides guidance for AI coding agents working on the **MyST-Parser** repository.

## Project Overview

MyST-Parser is a Sphinx extension and docutils parser for the MyST (Markedly Structured Text) Markdown flavor. It provides:

- An extended [CommonMark](https://commonmark.org)-compliant parser using [`markdown-it-py`](https://markdown-it-py.readthedocs.io/)
- A [docutils](https://docutils.sourceforge.io/) renderer that converts markdown-it tokens to docutils nodes
- A [Sphinx](https://www.sphinx-doc.org) extension for using MyST Markdown in Sphinx documentation

MyST is designed for technical documentation and publishing, offering a rich and extensible flavor of Markdown with support for roles, directives, and cross-references.

## Repository Structure

```
pyproject.toml          # Project configuration and dependencies
tox.ini                 # Tox test environment configuration

myst_parser/            # Main source code
├── __init__.py         # Package init with Sphinx setup() entry point
├── config/             # Configuration dataclasses
│   ├── main.py         # MdParserConfig dataclass
│   └── dc_validators.py # Dataclass field validators
├── parsers/            # Parser implementations
│   ├── sphinx_.py      # Sphinx parser (MystParser)
│   ├── docutils_.py    # Docutils parser and CLI tools
│   ├── mdit.py         # markdown-it-py setup and plugins
│   ├── directives.py   # Directive parsing utilities
│   └── options.py      # Option parsing for directives
├── mdit_to_docutils/   # Token-to-docutils rendering
│   ├── base.py         # DocutilsRenderer (main renderer)
│   ├── sphinx_.py      # SphinxRenderer (Sphinx-specific)
│   ├── transforms.py   # Docutils transforms
│   └── html_to_nodes.py # HTML-to-docutils conversion
├── sphinx_ext/         # Sphinx extension components
│   ├── main.py         # setup_sphinx() and config creation
│   ├── directives.py   # Custom Sphinx directives
│   ├── myst_refs.py    # Reference resolver post-transform
│   └── mathjax.py      # MathJax configuration
├── inventory.py        # Sphinx inventory file handling
├── mocking.py          # Mock objects for directive/role parsing
├── warnings_.py        # Warning types (MystWarnings enum)
├── cli.py              # Command-line interface
└── _compat.py          # Python version compatibility

tests/                  # Test suite
├── test_sphinx/        # Sphinx integration tests
│   ├── sourcedirs/     # Test documentation projects
│   └── test_sphinx_builds.py
├── test_renderers/     # Renderer unit tests
├── test_commonmark/    # CommonMark compliance tests
├── test_html/          # HTML output tests
├── test_docutils.py    # Docutils parser tests
└── test_anchors.py     # Heading anchor tests

docs/                   # Documentation source (MyST Markdown)
├── conf.py             # Sphinx configuration
├── index.md            # Documentation index
├── syntax/             # Syntax reference documentation
├── develop/            # Developer documentation
└── faq/                # FAQ and troubleshooting
```

## Development Commands

All commands should be run via [`tox`](https://tox.wiki) for consistency. The project uses `tox-uv` for faster environment creation.

### Testing

```bash
# Run all tests
tox

# Run a specific test file
tox -- tests/test_docutils.py

# Run a specific test function
tox -- tests/test_docutils.py::test_function_name

# Run tests with a specific Python/Sphinx version
tox -e py311-sphinx8

# Run with coverage
tox -- --cov=myst_parser

# Update regression test fixtures (this will initially produce an error code if the files change)
# but note, these files must pass for all python/sphinx/docutils versions
tox -- --regen-file-failure --force-regen
```

### Documentation

```bash
# Build docs (clean)
tox -e docs-clean

# Build docs (incremental)
tox -e docs-update

# Build with a specific builder (e.g., linkcheck)
BUILDER=linkcheck tox -e docs-update
```

### Code Quality

```bash
# Type checking with mypy
tox -e mypy

# Linting with ruff (auto-fix enabled)
tox -e ruff-check

# Formatting with ruff
tox -e ruff-fmt

# Run pre-commit hooks on all files
pre-commit run --all-files
```

## Code Style Guidelines

- **Formatter/Linter**: Ruff (configured in `pyproject.toml`)
- **Type Checking**: Mypy with strict settings (configured in `pyproject.toml`)
- **Pre-commit**: Use pre-commit hooks for consistent code style

### Best Practices

- **Type annotations**: Use complete type annotations for all function signatures. Use `TypedDict` for structured dictionaries, dataclasses for configuration.
- **Docstrings**: Use Sphinx-style docstrings (`:param:`, `:return:`, `:raises:`). Types are not required in docstrings as they should be in type hints.
- **Function Signatures**: Use `/` and `*` to enforce positional-only and keyword-only arguments where appropriate
- **Pure functions**: Where possible, write pure functions without side effects.
- **Error handling**: Use `MystWarnings` enum for warning types. Use `create_warning()` for user-facing warnings.
- **Testing**: Write tests for all new functionality. Use `pytest-regressions` for output comparison tests.

### Docstring Example

```python
def parse_directive_text(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [executablebooks/MyST-Parser](https://github.com/executablebooks/MyST-Parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
