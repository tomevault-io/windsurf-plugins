---
trigger: always_on
description: This file provides guidance to LLM agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to LLM agents when working with code in this repository.

---

## Project Summary

`sieves` is a Python library for rapid, production-minded prototyping of NLP pipelines with zero- and few-shot models and structured outputs. It provides:

- A document-centric pipeline (`Pipeline`) with caching and serialization
- Preprocessing tasks (document ingestion, text chunking)
- Predictive tasks (classification, NER, information extraction, relation extraction, QA, summarization, translation, sentiment analysis, PII masking)
- A unified model interface over structured-generation frameworks (Outlines, DSPy, Instructor, LangChain, Transformers, Ollama, GLiNER, etc.)
- Postprocessing and model distillation helpers

Key packages and concepts: `sieves.data.Doc`, `sieves.pipeline.Pipeline`, `sieves.tasks.*`, `sieves.model_wrappers.*`, `sieves.serialization.Config`.

### Objectives

- Provide reliable, structured outputs with zero/few-shot models
- Make pipelines easy to compose, observe, cache, and serialize
- Support multiple structured-generation model libraries behind one interface
- Enable distillation to smaller, local models for cost/performance optimization
- **Automated Prompt Construction**: Built-in XML generation for few-shot examples and standardized prompt formatting across all tasks.
- **Deep Model Inspection**: Recursive mapping of nested Pydantic models to DSPy signatures, ensuring full visibility of nested metadata like confidence scores.

---

## Repository Layout

```
sieves/                    # Core library
├── data/                  # Document model (Doc class)
├── pipeline/              # Pipeline orchestration and caching
├── tasks/                 # Preprocessing, predictive, and postprocessing tasks
│   ├── predictive/        # 9 task types (classification, NER, IE, RE, etc.)
│   └── preprocessing/     # Ingestion and chunking
├── model_wrappers/        # Unified interface to structured generation backends
└── serialization/         # Config and persistence helpers
docs/                      # MkDocs documentation
sieves/tests/              # Comprehensive test suite
pyproject.toml             # Dependencies, metadata, tool config
mkdocs.yml                 # Documentation build config
AGENTS.md                  # This file (Claude Code guidelines)
```

---

## Installation & Setup

**Python requirement:** 3.12 or 3.13 (3.14 not supported yet due to dependency wheels missing)

### Using `uv` (preferred)

```bash
uv sync                              # Base installation
uv sync --extra distill              # Add distillation (SetFit, Model2Vec)
uv sync --extra ingestion            # Add document parsing (Docling, Marker, NLTK)
uv sync --all-extras                 # Everything (includes test tools)
```

**Note:** As of recent updates, all supported model libraries (Outlines, DSPy, LangChain, Transformers, GLiNER2) are now core dependencies included in the base installation.

### Using pip (editable)

```bash
pip install -e .                     # Base
pip install -e ".[distill,ingestion]"  # With extras
```

### Environment Variables

Set only what you need:

```bash
OPENAI_API_KEY                       # For OpenAI models
ANTHROPIC_API_KEY                    # For Claude models
OPENROUTER_API_KEY                   # For OpenRouter
OLLAMA_HOST                          # For local Ollama models (e.g., http://localhost:11434)
```

---

## Development Commands

All commands assume `uv` is available; adjust for `pip` + venv if needed.

### Verification & Quality

```bash
# Install development dependencies (includes test tools)
uv sync --all-extras

# Run type checker (strict mode)
uv run mypy sieves

# Run linter
uv run ruff check .

# Auto-fix safe lint issues
uv run ruff check . --fix

# Format code
uv run black .

# Run full quality pipeline
uv run mypy sieves && uv run ruff check . && uv run black --check .
```

### Testing

```bash
# Run all tests
uv run pytest -q

# Run with coverage report
uv run pytest --cov=sieves

# Run a specific test file
uv run pytest sieves/tests/test_doc.py -v

# Run tests matching a pattern (e.g., classification tasks)
uv run pytest -k classification -v

# Run fast tests only (skip slow tests)
uv run pytest -m "not slow"
```

### Documentation

```bash
# Build and serve docs locally
uv run mkdocs serve

# Build static docs
uv run mkdocs build --strict
```

### Import & Sanity Check

```bash
# Verify installation
uv run python -c "import sieves; print(sieves.__name__)"
```

---

## Architecture Overview

### Core Abstractions

1. **Doc** (`sieves.data.Doc`)
   - Container for text, URI, chunks, images, and processing results
   - `results` dict stores task outputs keyed by task ID
   - `gold` dict stores ground truth data for evaluation
   - Auto-chunks text on initialization using Chonkie
   - Supports image inputs via PIL (stored in `images` field)

2. **Pipeline** (`sieves.pipeline.Pipeline`)
   - Orchestrates sequential task execution
   - Caches by document hash (text or URI)
   - Supports composition via `+` operator
   - Supports evaluation via `evaluate(docs)`
   - Serializable via `dump()`/`load()`

3. **Task** (`sieves.tasks.core.Task`)
   - Base class for all processing steps

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MantisAI/sieves](https://github.com/MantisAI/sieves) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
