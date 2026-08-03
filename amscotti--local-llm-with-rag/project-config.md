---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an experimental sandbox for **agentic RAG** (Retrieval-Augmented Generation) using local LLMs via Ollama and Pydantic AI. Unlike fixed RAG pipelines, the AI agent decides when and how to search documents.

**Key constraint:** Requires Ollama running locally at `http://localhost:11434`.

## Commands

```bash
# Install dependencies
uv sync

# Run the app
uv run streamlit run interfaces/streamlit_app.py

# Code quality
uv run ruff check .           # Lint
uv run ruff check . --fix     # Auto-fix
uv run ruff format .          # Format
uv run pyrefly check          # Type check

# Benchmark models (requires Ollama running)
uv run python -m bench --models qwen3:8b qwen3:14b qwen3.5:9b --judge qwen3-coder:480b-cloud
uv run python -m bench --limit 2 -v          # quick smoke test (2 questions)
uv run python -m bench --skip-run             # re-judge existing raw outputs
uv run python -m bench --max-tool-calls 3     # tighten search loop protection
```

## Architecture

```
interfaces/streamlit_app.py ──▶ core/agent.py (ResearchAgent) ──▶ Ollama LLM
                                        │
                                        │ @agent.tool
                                        ▼
                                 core/document_loader.py ──▶ LanceDB (storage/)
                                        │
                                        ▼
                                 Ollama Embeddings (nomic-embed-text)
```

**Core flow:**
1. Documents loaded from folder into LanceDB vector store (supports PDF, Word, PowerPoint, Excel, Markdown, HTML, CSV, JSON via MarkItDown)
2. ResearchAgent uses Pydantic AI with a `search_documents` tool
3. Agent autonomously decides whether to search documents or answer directly
4. Streaming responses via `agent.run_stream_sync()`

## Project Structure

```
├── core/
│   ├── __init__.py
│   ├── agent.py                  # Pydantic AI agent with RAG tool
│   ├── document_loader.py        # Document loading, LanceDB integration
│   └── models.py                 # Ollama model management
├── interfaces/
│   ├── __init__.py
│   └── streamlit_app.py          # Streamlit web interface (main entry point)
├── bench/                        # Model benchmarking harness
│   ├── questions.jsonl           # Eval questions w/ golden answers + key facts
│   ├── runner.py                 # Runs a model via ResearchAgent, captures metrics
│   ├── judge.py                  # LLM-as-judge (Ollama) scoring vs golden answers
│   └── cli.py                    # CLI orchestrator -> results/scores.csv + summary.md
├── Research/                     # Sample documents
└── storage/                      # LanceDB vector store (gitignored)
```

## Benchmarking

The `bench/` package compares LLM models on the agentic RAG pipeline. It runs each
candidate model against `bench/questions.jsonl` (10 questions spanning factual,
methodology, numeric, and comparison categories) and uses a larger **judge** model
via Ollama to score answers 1-5 against hand-written golden answers + key facts.

**Captured per (model, question):** answer text, number of `search_documents` calls,
total time, time-to-first-text, judge score, covered facts.

**Outputs** (in `bench/results/`, gitignored):
- `raw/<model>.jsonl` — full raw outputs per model
- `scores.csv` — one row per (model, question)
- `summary.md` — per-model averages + per-question score table

Edit `bench/questions.jsonl` to add questions; each line needs `id`, `category`,
`question`, `reference_answer`, and `key_facts` (a list the judge checks for).

## Tech Stack

- **Pydantic AI**: Agent orchestration, tool calling, streaming
- **LanceDB**: Vector store with native Ollama embeddings
- **MarkItDown**: Document loading (PDF, Word, PowerPoint, Excel, etc.)
- **Ollama**: Local LLM and embeddings
- **Streamlit**: Web interface

## Model Notes

- Default LLM: `qwen3.5:9b`
- Default embeddings: `nomic-embed-text` (768 dimensions)
- Recommended models: `qwen3.5:9b` (best overall), `qwen3:8b` (strong alternative)
- Judge model for benchmarks: `qwen3-coder:480b-cloud` (free-tier Ollama cloud)
- Some models don't support tool calling in Ollama - test before using for this project
- Hybrid/MoE models with few active parameters (e.g. `lfm2.5:8b-a1b`) are unreliable for agentic RAG
- Vector store only reloads when folder path changes (not on model switch)

## Code Style Guidelines

### Naming Conventions
- Functions and variables: `snake_case`
- Classes: `PascalCase`
- Constants: `UPPER_CASE`
- Private functions: `_single_leading_underscore`

### Import Organization
Group imports in this order (separated by blank lines):
1. Standard library imports
2. Third-party imports
3. Local application imports

```python
import logging
import os
from pathlib import Path
from typing import Annotated

import lancedb
from pydantic_ai import Agent, RunContext

from core.document_loader import load_documents_into_database
```

### Type Hints
- Always include type hints for function parameters and return values
- Use `-> None` for functions that don't return a value

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amscotti/local-LLM-with-RAG](https://github.com/amscotti/local-LLM-with-RAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
