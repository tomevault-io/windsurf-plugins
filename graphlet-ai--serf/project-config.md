---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with the SERF: Semantic Entity Resolution Framework in this repository. See @README.md for general project information.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with the SERF: Semantic Entity Resolution Framework in this repository. See @README.md for general project information.

## Commands

### Development

- Install Dependencies: `uv sync`
- Run CLI: `uv run serf`
- Test all: `uv run pytest tests/`
- Test single: `uv run pytest tests/path_to_test.py::test_name`
- Lint: `uv run ruff check src tests`
- Format: `uv run ruff format src tests`
- Lint + Fix: `uv run ruff check --fix src tests`
- Type check: `uv run zuban check src tests`
- Pre-commit: `pre-commit run --all-files`

### Docker Development (via Taskfile)

- Build: `docker compose build`
- Run: `docker compose up -d`
- Stop: `docker compose down`
- Tail Logs: `docker compose logs -f`

### Common Workflows

- None yet

## Architecture Overview

### Project Structure

- **src/** - Source code
  - **serf/** - Core application code
    - **block/** - Blocking module - semantic clustering using sentence-transformers
    - **match/** - Matching module - matching entire blocks at once with Gemini models
    - **merge/** - Merging module - Record and field-level merging utilities
    - **edge/** - Edge resolution module - deduplication of edges after node merges
  - **baml_src/** - BAML templates for LLM extraction
- **data/** - Default data storage directory
- **tests/** - Test suite

### Key Technologies

- **LLM Integration**: BAML (Boundary AI Markup Language) for structured extraction
- **DSPy**: Programming—not prompting—LMs - a framework for building and optimizing LLM pipelines. See the Project's @assets/DSPy.md [DSPy Programming Guide](assets/DSPy.md) and read the docs at [DSPy Documentation](https://dspy.ai/api/).
- **Sentence Transformers**: A library for state-of-the-art sentence embeddings
- **Qwen3 Embeddings**: Top MTEB leaderboard embedding across most categories.
- **Gemini Models**: Advanced models for matching and merging entities
- **Data Processing**: Apache Spark (PySpark) for ETL and graph operations

### Data Flow

1. **Blocking**: Group articles into smaller groups to reduce the number of pairwise comparisons at quadratic complexity.
2. **Schema Alignment**: Align schemas between different data sources to ensure consistency.
3. **Matching + Merging**: Match and merge entities across different sources.
4. **Edge Resolution**: Deduplicate edges after node merges.

## Code Style

- KISS: KEEP IT SIMPLE STUPID. Do not over-engineer solutions. ESPECIALLY for Spark / PySpark.
- Line length: 100 characters
- Python version: 3.12
- Formatter: Ruff (replaces black + isort + flake8)
- Types: Always use type annotations, warn on any return
- Imports: Use absolute imports, organize imports with Ruff isort
- Error handling: Use specific exception types with logging
- Naming: snake_case for variables / functions, CamelCase for classes
- DSPy: Use DSPy signatures for all LLM-related code
- Whitespaces: leave no trailing whitespaces, use 4 spaces for indentation, leave no whitespace on blank lines
- Blank lines: Do not indent any blank lines in Python files. Indent should be 0 for these lines. Indent to 0 spaces when replacing a line with a blank line.
- Strings: Use double quotes for strings, use f-strings for string interpolation
- Docstrings: Use Numpy style for docstrings, include type hints in docstrings
- Comments: Use comments to explain complex code, avoid obvious comments
- Tests: Use pytest for testing, include type hints in test functions, use fixtures for setup/teardown
- Tests: Don't make a class to contain unit tests. Just write the tests in pytest style.
- Type hints: Use Python 3.9 type hints for all function parameters and return types. Use `list`, `dict`, `tuple`, etc. instead of `List`, `Dict`, `Tuple` from the `typing` module. Use `Optional` from the `typing` module for optional parameters.
- Type checking: Use zuban for type checking, run zuban before committing code. It is mypy compatible.
- Logging: Use logging for error handling, avoid print statements. Always use `from serf.logs import get_logger` and `logger = get_logger(__name__)`
- Documentation: Use Sphinx for documentation, include docstrings in all public functions/classes
- Code style: Follow PEP 8 for Python code style, use Ruff for linting and formatting
- Zuban: Use zuban for type checking, run zuban before committing code. Configure it in `pyproject.toml`.
- Pre-commit: Use pre-commit for linting and formatting, configure it in `.pre-commit-config.yaml`
- Git: Use git for version control, commit often with clear messages, use branches for new features/bug fixes. Always test new features in the CLI before you commit them.
- uv: Use uv for dependency management and packaging, configure it in `pyproject.toml`
- discord.py package - always use selective imports for `discord` - YES `from discord import x` - NO `import discord`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Graphlet-AI/serf](https://github.com/Graphlet-AI/serf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
