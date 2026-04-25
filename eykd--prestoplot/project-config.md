---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

PrestoPlot is a Python library and CLI tool for text generation using YAML-based generative grammars. It supports idea generation, name generation, and story creation through a template system inspired by Tracery.

## Development Commands

### Testing
- Run tests: `./runtests.sh` (uses uv and pytest with coverage)
- Single test: `uv run python -m pytest tests/test_specific.py`
- Test with tox: `tox` (tests across multiple Python versions)

### Linting and Code Quality
- Lint code: `uv run ruff check src/ tests/`
- Format code: `uv run ruff format src/ tests/`
- Type checking: Not configured (but we do use type annotations, lightly checked by ruff)

### Building and Installation
- Install in development mode: `uv sync --all-groups`
- Build package: `uv build`
- Run CLI: `uv run presto --help`

## Core Architecture

### CLI Entry Point
- Main CLI script: `presto` command (defined in `src/prestoplot/cli.py`)
- Two main commands:
  - `presto run <file.yaml>`: Generate text from YAML grammar files
  - `presto http <file.yaml>`: Serve HTTP endpoint for text generation

### Core Components

**Storage Layer** (`storages.py`):
- `FileStorage`: Loads YAML grammar files from filesystem
- `CompilingFileStorage`: Caches compiled grammars as MessagePack files

**Grammar Processing** (`grammars.py`):
- Parses YAML grammar files with `include` support
- Supports f-string and Jinja2 template rendering
- Handles recursive grammar includes

**Story Rendering** (`story.py`):
- Main entry point: `render_story(storage, name, start="Begin", seed=None, **kwargs)`
- Requires `Begin:` stanza in grammar files

**Context Management** (`contexts.py`):
- Manages random seeds and generation context
- Provides deterministic generation with same seeds

**Text Processing** (`texts.py`):
- Template rendering engines (f-string and Jinja2)
- Text generation utilities

**Additional Features**:
- `markov.py`: Markov chain text generation
- `db.py`: Database utilities for grammar storage
- `http.py`: HTTP server for web-based generation

### Grammar File Format
- YAML-based with Python f-string syntax
- Required `Begin:` stanza as entry point
- Support for `include:` to compose grammar files
- Modes: `reuse` (default), `pick` (no replacement), `markov` (chain generation)
- Seeded generation using keys (e.g., `{Name.character1}`)

## Test Structure
- Unit tests in `tests/` directory
- Test data in `tests/data/` (sample YAML grammars)
- Coverage reporting enabled by default


## When asked to create new conventions

When asked to create a new convention (`CLAUDE.md`), add a second-level
heading section to this document, `CLAUDE.md`.

* Name the new convention heading with a short, descriptive title.
* Use the first line of the section to elaborate on the "When..." of the heading.
* Use bullet points to organize further details for the convention.
* Use full imperative sentences.
* Keep new conventions short and to the point.
* Use short examples for complex conventions.

## Python code style and quality

When writing or editing Python code (`*.py`), follow these quality standards:

* Use PEP8 style with CamelCase for class names and snake\_case for variables/functions.
* Include type annotations for all functions, methods, and complex structures.
* Add Google Style docstrings to all packages, modules, functions, classes, and methods.
* Run code quality tools:

  * Format: `uv run ruff format`
  * Lint: `uv run ruff check --fix`


## Testing

When writing Python code (`*.py`), follow these testing practices:

* Write tests first for each change using pytest.
* Organize tests in a dedicated `tests/` folder in the project root.
* Name test files by package and module, omitting the root `cloudmarch` package name.

  * Example: `tests/test_config_loader.py` tests `src/cloudmarch/config/loader.py`
* Use descriptive names for test functions and methods.
* Group related tests in test classes.
* Use fixtures for complex setup.
* Aim for 100% test coverage for code under `src/`.
* When writing tests, move common fixtures to `tests/conftest.py`.
* Run tests with `./scripts/runtests.sh` (which accepts normal `pytest` arguments and flags).

  * Example: `./scripts/runtests.sh tests/test_config_loader.py`

## Test organization with classes

When organizing tests in pytest, group related tests using `TestX` classes:

* Use `TestX` classes to group tests for the same module, function, or behavior.
* Name test classes with descriptive titles like `TestGrammarParser` or `TestFileStorage`.
* Do not inherit from `unittest.TestCase` since pytest handles plain classes.
* Place setup and teardown logic in `setup_method` and `teardown_method`.
* Example:
  ```python
  class TestGrammarParser:
      @pytest.fixture
      def parser(self) -> GrammarParser:
          return GrammarParser()

      def test_parses_simple_grammar(self, parser: GrammarParser) -> None:
          result = parser.parse("Begin: hello")
          assert result["Begin"] == ["hello"]
  ```

## Unit testing with pytest

When writing unit tests for Python libraries, follow these pytest best practices:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eykd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
