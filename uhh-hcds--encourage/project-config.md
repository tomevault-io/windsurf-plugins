---
trigger: always_on
description: EncouRAGe is a Python library for running and evaluating Retrieval-Augmented Generation (RAG) methods. The library provides modular components for RAG pipelines including vector databases (ChromaDB, Qdrant), LLM inference (OpenAI), template engines (Jinja2), and comprehensive evaluation metrics.
---

# EncouRAGe - Copilot Instructions

## Repository Overview

EncouRAGe is a Python library for running and evaluating Retrieval-Augmented Generation (RAG) methods. The library provides modular components for RAG pipelines including vector databases (ChromaDB, Qdrant), LLM inference (OpenAI), template engines (Jinja2), and comprehensive evaluation metrics.

**Repository Statistics:**
- Languages: Python (3.12+)
- Size: Medium (~100-200 source files)
- Type: Python library/package
- Package Name: `encourage-rag`
- Build System: `uv` (modern Python package manager)
- Test Framework: pytest
- Linting: ruff, isort

## Development Environment Setup

### Initial Setup
1. **Clone and install dependencies:**
   ```bash
   # Always use uv for dependency management
   uv sync
   
   # For development with all extras (including Jupyter):
   uv sync --all-extras
   ```

2. **Environment Requirements:**
   - Python 3.12+ required (3.13 also supported)
   - Virtual environment is mandatory for Makefile commands
   - `uv` package manager must be installed (version 0.4.28)

### Important Notes
- Always run `uv sync` before building or testing to ensure dependencies are up-to-date
- The Makefile requires running inside a virtual environment (or Conda, with a warning)
- Pre-commit hooks use ruff for formatting and linting

## Build and Validation

### Linting
```bash
# Run linter (this also installs uv if needed)
make lint

# This runs:
# - ruff check src/encourage src/tests
# - uvx ty check (type checking)
```

### Formatting
```bash
# Auto-format code
make format

# This runs:
# - isort src/encourage src/tests
# - ruff format src/encourage src/tests
```

### Testing
```bash
# Run all tests
make tests

# Run only last failed tests
make tests-last-failed

# Direct pytest (ensure uv sync ran first):
uv run pytest
```

**Test Configuration:**
- Test directory: `src/tests/`
- pytest config in `pyproject.toml`
- Tests run on Python 3.9, 3.10, 3.11, 3.12 in CI
- Warnings are filtered (see pyproject.toml for specifics)

## Code Style Guidelines

### Linting Rules (ruff)
- Line length: 100 characters
- Selected rules: `E`, `F`, `W`, `I`, `D`, `A`, `N`, `B`, `SIM`, `C4`, `TID`
- Ignored rules: `E741`, `D213`, `D105`, `D107`, `D203`, `D401`, `D407`, `D406`, `D106`, `B006`, `B008`, `B905`

### Import Rules
- **Absolute imports only** - relative imports are banned
- Use `isort` with black profile
- Import sorting is enforced via pre-commit hooks

### File-Specific Rules
- `test_*.py`: Ignores `D` (docstrings) and `E402` (module level import order)
- `__init__.py`: Ignores `D` (docstrings)
- `*.ipynb` (Jupyter notebooks): Excluded from ruff formatting

## Continuous Integration

### GitHub Actions Workflow
- File: `.github/workflows/lint_test.yml`
- Triggers: Push to main, all pull requests
- Matrix: Python 3.9, 3.10, 3.11, 3.12

**CI Steps:**
1. Checkout code
2. Setup Python
3. Install uv and activate environment
4. Run `uv sync --all-extras`
5. Run ruff-action (linting)
6. Run `uv run pytest` (testing)

**Important:** All PRs must pass linting and tests on all Python versions in the CI matrix.

**Note:** There's a discrepancy between the CI matrix (3.9-3.12) and pyproject.toml requirement (>=3.12). The package officially requires Python 3.12+, but CI currently tests older versions.

### Pre-commit Hooks
- Configuration: `.pre-commit-config.yaml`
- Runs: `ruff check --fix && ruff format`
- Install with: `pre-commit install`

## Project Structure

### Source Code Layout
```
src/
├── encourage/              # Main package
│   ├── __init__.py
│   ├── handler/           # Request/response handlers
│   ├── llm/              # LLM inference (OpenAI integration)
│   ├── metrics/          # Evaluation metrics
│   ├── prompts/          # Jinja2 templates
│   ├── rag/              # RAG method implementations
│   ├── utils/            # Utility functions
│   └── vector_store/     # ChromaDB & Qdrant integrations
└── tests/                # Test suite (mirrors src structure)
    ├── conftest.py       # pytest configuration
    ├── fake_responses.py # Test fixtures
    ├── metrics/
    ├── prompts/
    ├── rag/
    └── vector_store/
```

### Configuration Files
- `pyproject.toml` - Package metadata, dependencies, tool configs (ruff, isort, pytest, coverage)
- `Makefile` - Build automation (format, lint, test, sync)
- `uv.lock` - Locked dependency versions (551KB)
- `.pre-commit-config.yaml` - Pre-commit hook configuration
- `.github/workflows/` - CI/CD pipelines
- `.github/dependabot.yml` - Dependency update automation

### Key Dependencies
- **LLM Inference:** openai (>=1.98.0)
- **Vector Databases:** chromadb (>=1.0.10), qdrant-client
- **Evaluation:** evaluate (>=0.4.3), bert-score, rouge-score, ir-measures
- **Templates:** jinja2
- **ML Tracking:** mlflow (>=2.18.0)
- **NLP:** sentence-transformers (>=3.2.1), nltk (>=3.9.1)

## Common Workflows

### Making Changes
1. Create feature branch: `git checkout -b feature/your-feature`
2. Install dependencies: `uv sync` (or `uv sync --all-extras` for dev)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uhh-hcds/encourage](https://github.com/uhh-hcds/encourage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
