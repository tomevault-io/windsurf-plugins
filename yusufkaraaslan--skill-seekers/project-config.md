---
trigger: always_on
description: Comprehensive reference for AI coding agents. Skill Seekers is a Python CLI tool (v3.5.0) that converts documentation sites, GitHub repos, PDFs, videos, notebooks, wikis, and more into AI-ready skills for 21+ LLM platforms and RAG pipelines.
---

# AGENTS.md - Skill Seekers

Comprehensive reference for AI coding agents. Skill Seekers is a Python CLI tool (v3.5.0) that converts documentation sites, GitHub repos, PDFs, videos, notebooks, wikis, and more into AI-ready skills for 21+ LLM platforms and RAG pipelines.

## Project Overview

**Skill Seekers** is a universal preprocessing layer that transforms raw documentation and code into structured knowledge assets. It supports 17+ source types and exports to 21+ AI platforms including Claude, Gemini, OpenAI, LangChain, LlamaIndex, and various vector databases.

### Key Capabilities
- **Source Types (17):** Documentation websites, GitHub repos, PDFs, Word docs, EPUBs, videos, local codebases, Jupyter notebooks, HTML, OpenAPI specs, AsciiDoc, PowerPoint, Confluence, Notion, RSS feeds, man pages, chat exports
- **Export Targets (21):** Claude, Gemini, OpenAI, MiniMax, OpenCode, Kimi, DeepSeek, Qwen, OpenRouter, Together AI, Fireworks AI, Markdown, LangChain, LlamaIndex, Haystack, Weaviate, ChromaDB, FAISS, Qdrant, Pinecone
- **MCP Server:** FastMCP-based Model Context Protocol server for AI assistant integration

## Setup

```bash
# REQUIRED before running tests (src/ layout — tests hard-exit if package not installed)
pip install -e .

# With dev tools (pytest, ruff, mypy, coverage)
pip install -e ".[dev]"

# With specific LLM platform support
pip install -e ".[gemini]"      # Google Gemini
pip install -e ".[openai]"      # OpenAI ChatGPT
pip install -e ".[all-llms]"    # All LLM platforms

# With all optional dependencies (except video-full)
pip install -e ".[all]"

# Full video processing (heavy dependencies)
pip install -e ".[video-full]"
```

Note: `tests/conftest.py` checks that `skill_seekers` is importable and calls `sys.exit(1)` if not. Always install in editable mode first.

### Environment Variables

Create a `.env` file or export these variables:
```bash
ANTHROPIC_API_KEY      # For Claude AI enhancement
GOOGLE_API_KEY         # For Gemini support
OPENAI_API_KEY         # For OpenAI support
GITHUB_TOKEN           # For GitHub repo scraping (higher rate limits)
```

## Build / Test / Lint Commands

```bash
# Run ALL tests (never skip tests — all must pass before commits)
pytest tests/ -v

# Run a single test file
pytest tests/test_scraper_features.py -v

# Run a single test function
pytest tests/test_scraper_features.py::test_detect_language -v

# Run a single test class method
pytest tests/test_adaptors/test_claude_adaptor.py::TestClaudeAdaptor::test_package -v

# Skip slow/integration tests
pytest tests/ -v -m "not slow and not integration"

# With coverage
pytest tests/ --cov=src/skill_seekers --cov-report=term

# Lint (ruff)
ruff check src/ tests/
ruff check src/ tests/ --fix

# Format (ruff)
ruff format --check src/ tests/
ruff format src/ tests/

# Type check (mypy)
mypy src/skill_seekers --show-error-codes --pretty
```

**Pytest config** (from pyproject.toml): `addopts = "-v --tb=short --strict-markers"`, `asyncio_mode = "auto"`, `asyncio_default_fixture_loop_scope = "function"`.

**Test markers:** `slow`, `integration`, `e2e`, `venv`, `bootstrap`, `benchmark`, `asyncio`.

**Async tests:** use `@pytest.mark.asyncio`; asyncio_mode is `auto` so the decorator is often implicit.

**Test count:** 160 test files (138 in `tests/`, 22 in `tests/test_adaptors/`).

## Code Style

### Formatting Rules (ruff — from pyproject.toml)
- **Line length:** 100 characters
- **Target Python:** 3.10+
- **Enabled lint rules:** E, W, F, I, B, C4, UP, ARG, SIM
- **Ignored rules:** E501 (line length handled by formatter), F541 (f-string style), ARG002 (unused method args for interface compliance), B007 (intentional unused loop vars), I001 (formatter handles imports), SIM114 (readability preference)

### Imports
- Sort with isort (via ruff); `skill_seekers` is first-party
- Standard library → third-party → first-party, separated by blank lines
- Use `from __future__ import annotations` only if needed for forward refs
- Guard optional imports with try/except ImportError (see `adaptors/__init__.py` pattern):
  ```python
  try:
      from .claude import ClaudeAdaptor
      from .minimax import MiniMaxAdaptor
  except ImportError:
      ClaudeAdaptor = None
      MiniMaxAdaptor = None
  ```

### Naming Conventions
- **Files:** `snake_case.py` (e.g., `source_detector.py`, `config_validator.py`)
- **Classes:** `PascalCase` (e.g., `SkillAdaptor`, `ClaudeAdaptor`, `SourceDetector`)
- **Functions/methods:** `snake_case` (e.g., `get_adaptor()`, `detect_language()`)
- **Constants:** `UPPER_CASE` (e.g., `ADAPTORS`, `DEFAULT_CHUNK_TOKENS`, `VALID_SOURCE_TYPES`)
- **Private:** prefix with `_` (e.g., `_read_existing_content()`, `_validate_unified()`)

### Type Hints
- Gradual typing — add hints where practical, not enforced everywhere
- Use modern syntax: `str | None` not `Optional[str]`, `list[str]` not `List[str]`
- MyPy config: `disallow_untyped_defs = false`, `check_untyped_defs = true`, `ignore_missing_imports = true`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yusufkaraaslan/Skill_Seekers](https://github.com/yusufkaraaslan/Skill_Seekers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
