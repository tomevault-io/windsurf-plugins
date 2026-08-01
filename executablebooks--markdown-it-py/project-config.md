---
trigger: always_on
description: Manages parsing rules:
---

# AGENTS.md

This file provides guidance for AI coding agents working on the **markdown-it-py** repository.

## Project Overview

markdown-it-py is a Python port of [markdown-it](https://github.com/markdown-it/markdown-it), the JavaScript Markdown parser. It provides:

- A Markdown parser following the [CommonMark spec](https://commonmark.org/)
- Configurable syntax: you can add new rules and even replace existing ones
- Pluggable architecture with support for syntax extensions (see [mdit-py-plugins](https://github.com/executablebooks/mdit-py-plugins))
- High performance with efficient parsing algorithms
- Safe by default with configurable HTML handling

markdown-it-py is designed as a foundation for projects requiring robust Markdown parsing in Python, with the same design principles as the original JavaScript implementation.

## Repository Structure

```
pyproject.toml          # Project configuration and dependencies (flit)
tox.ini                 # Tox test environment configuration (use with tox-uv for faster env creation)

markdown_it/            # Main source code
├── __init__.py         # Package init
├── main.py             # MarkdownIt main class
├── token.py            # Token dataclass
├── ruler.py            # Ruler class for managing rules
├── tree.py             # SyntaxTreeNode for AST representation
├── renderer.py         # RendererHTML and RendererProtocol
├── parser_core.py      # ParserCore - top-level rules executor
├── parser_block.py     # ParserBlock - block-level tokenizer
├── parser_inline.py    # ParserInline - inline tokenizer
├── utils.py            # Utility types (OptionsType, PresetType, etc.)
├── common/             # Common utilities
├── helpers/            # Helper functions
├── presets/            # Configuration presets (commonmark, gfm-like, zero, etc.)
├── rules_core/         # Core parsing rules
├── rules_block/        # Block-level parsing rules
├── rules_inline/       # Inline parsing rules
├── cli/                # Command-line interface
└── py.typed            # PEP 561 marker

tests/                  # Test suite
├── test_api/           # API tests
├── test_cmark_spec/    # CommonMark spec compliance tests
├── test_port/          # Port-specific tests
├── test_tree/          # SyntaxTreeNode tests
├── fuzz/               # Fuzzing tests for OSS-Fuzz
├── test_cli.py         # CLI tests
├── test_linkify.py     # Linkify tests
└── test_tree.py        # Tree tests

docs/                   # Documentation source
├── conf.py             # Sphinx configuration
├── index.md            # Documentation index
├── architecture.md     # Design principles
├── using.md            # Usage guide
├── plugins.md          # Plugin documentation
├── contributing.md     # Contributing guide
├── performance.md      # Performance benchmarks
└── security.md         # Security considerations

benchmarking/           # Performance benchmarking
scripts/                # Utility scripts
```

## Development Commands

All commands should be run via [`tox`](https://tox.wiki) for consistency. The project uses `tox-uv` for faster environment creation.

### Testing

```bash
# Run all tests
tox

# Run tests with specific Python version
tox -e py311

# Run tests with plugins
tox -e py311-plugins

# Run a specific test file
tox -- tests/test_api/test_main.py

# Run a specific test function
tox -- tests/test_api/test_main.py::test_get_rules

# Run tests with coverage
tox -- --cov=markdown_it --cov-report=html
```

### Documentation

```bash
# Build docs (clean)
tox -e docs-clean

# Build docs (incremental)
tox -e docs-update

# Specific builder (e.g., linkcheck)
BUILDER=linkcheck tox -e docs-update
```

### Benchmarking and Profiling

```bash
# Run core benchmarks
tox -e py311-bench-core

# Run package comparison benchmarks
tox -e py311-bench-packages

# Run profiler
tox -e profile
```

### Fuzzing

```bash
# Run fuzzer on testcase file
tox -e fuzz path/to/testcase
```

### Code Quality

```bash
# Run pre-commit hooks on all files
pre-commit run --all-files

# Type checking (via pre-commit)
pre-commit run mypy --all-files

# Linting and formatting (via pre-commit)
pre-commit run ruff --all-files
pre-commit run ruff-format --all-files
```

## Code Style Guidelines

- **Formatter/Linter**: Ruff (configured in `pyproject.toml`)
- **Type Checking**: Mypy with strict settings (configured in `pyproject.toml`)
- **Pre-commit**: Use pre-commit hooks for consistent code style (`.pre-commit-config.yaml`)

### Best Practices

- **Type annotations**: Use complete type annotations for all function signatures. The codebase uses strict mypy settings.
- **Docstrings**: Use Google-style or Sphinx-style docstrings. Types are not required in docstrings as they should be in type hints.
- **Pure functions**: Where possible, write pure functions without side effects.
- **Immutability**: Prefer immutable data structures. The `Token` class uses dataclass with appropriate mutability.
- **Testing**: Write tests for all new functionality. Use `pytest-regressions` for output comparison tests.

### Type Annotation Example

```python
from __future__ import annotations

from typing import Sequence

def parse_blocks(
    state: StateBlock,
    start_line: int,
    end_line: int,
    silent: bool = False
) -> bool:
    """Parse block-level content.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [executablebooks/markdown-it-py](https://github.com/executablebooks/markdown-it-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
