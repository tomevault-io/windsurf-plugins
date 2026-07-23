---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cognee is an open-source AI memory platform that transforms raw data into persistent knowledge graphs for AI agents. It replaces traditional RAG (Retrieval-Augmented Generation) with an ECL (Extract, Cognify, Load) pipeline combining vector search, graph databases, and LLM-powered entity extraction.

**Requirements**: Python 3.10 - 3.14

## Development Commands

### Setup
```bash
# Create virtual environment (recommended: uv)
uv venv && source .venv/bin/activate

# Install with pip, poetry, or uv
uv pip install -e .

# Install with dev dependencies
uv pip install -e ".[dev]"

# Install with specific extras
uv pip install -e ".[postgres,neo4j,docs,chromadb]"

# Set up pre-commit hooks
pre-commit install
```

### Available Installation Extras
- **postgres** / **postgres-binary** - PostgreSQL + PGVector support (also enables the Postgres session-cache backend, `CACHE_BACKEND=postgres`)
- **neo4j** - Neo4j graph database support
- **neptune** - AWS Neptune support
- **chromadb** - ChromaDB vector database
- **docs** - Document processing (unstructured library)
- **scraping** - Web scraping (Tavily, BeautifulSoup, Playwright)
- **langchain** - LangChain integration
- **llama-index** - LlamaIndex integration
- **anthropic** - Anthropic Claude models
- **gemini** - Google Gemini models
- **ollama** - Ollama local models
- **mistral** - Mistral AI models
- **groq** - Groq API support
- **llama-cpp** - Llama.cpp local inference
- **huggingface** - HuggingFace transformers
- **aws** - S3 storage backend
- **redis** - Redis caching
- **graphiti** - Graphiti-core integration
- **baml** - BAML structured output
- **dlt** - Data load tool (dlt) integration
- **docling** - Docling document processing
- **codegraph** - Code graph extraction
- **evals** - Evaluation tools
- **deepeval** - DeepEval testing framework
- **posthog** - PostHog analytics
- **tracing** - OpenTelemetry tracing
- **distributed** - Modal distributed execution
- **dev** - All development tools (pytest, ty, ruff, etc.)
- **debug** - Debugpy for debugging

### Testing
```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=cognee --cov-report=html

# Run specific test file
pytest cognee/tests/test_custom_model.py

# Run specific test function
pytest cognee/tests/test_custom_model.py::test_function_name

# Run async tests
pytest -v cognee/tests/integration/

# Run unit tests only
pytest cognee/tests/unit/

# Run integration tests only
pytest cognee/tests/integration/
```

### Code Quality
```bash
# Run ruff linter
ruff check .

# Run ruff formatter
ruff format .

# Run both linting and formatting (pre-commit)
pre-commit run --all-files

# Type checking with ty
ty check .
```

### Running Cognee
```bash
# Using Python SDK
uv run python examples/demos/simple_cognee_example.py

# Using CLI
cognee-cli add "Your text here"
cognee-cli cognify
cognee-cli search "Your query"
cognee-cli delete --all

# Launch full stack with UI
cognee-cli -ui
```

## Architecture Overview

### Core Workflow: add → cognify → search/memify

1. **add()** - Ingest data (files, URLs, text) into datasets
2. **cognify()** - Extract entities/relationships and build knowledge graph
3. **search()** - Query knowledge using various retrieval strategies
4. **memify()** - Enrich graph with additional context and rules

### Key Architectural Patterns

#### 1. Pipeline-Based Processing
All data flows through task-based pipelines (`cognee/modules/pipelines/`). Tasks are composable units that can run sequentially or in parallel. Example pipeline tasks: `classify_documents`, `extract_graph_from_data`, `add_data_points`.

#### 2. Interface-Based Database Adapters
Multiple backends are supported through adapter interfaces:
- **Graph**: Ladybug (default), Neo4j, Neptune, Postgres via `GraphDBInterface`
- **Vector**: LanceDB (default), ChromaDB, PGVector via `VectorDBInterface`
- **Relational**: SQLite (default), PostgreSQL

Key files:
- `cognee/infrastructure/databases/graph/graph_db_interface.py`
- `cognee/infrastructure/databases/vector/vector_db_interface.py`

#### 3. Multi-Tenant Access Control
User → Dataset → Data hierarchy with permission-based filtering. Enable with `ENABLE_BACKEND_ACCESS_CONTROL=True`. Each user+dataset combination can have isolated graph/vector databases (when using supported backends: Ladybug, LanceDB, SQLite, Postgres).

### Layer Structure

```
API Layer (cognee/api/v1/)
    ↓
Main Functions (add, cognify, search, memify)
    ↓
Pipeline Orchestrator (cognee/modules/pipelines/)
    ↓
Task Execution Layer (cognee/tasks/)
    ↓
Domain Modules (graph, retrieval, ingestion, etc.)
    ↓
Infrastructure Adapters (LLM, databases)
    ↓
External Services (OpenAI, Ladybug, LanceDB, etc.)
```

### Critical Data Flow Paths

#### ADD: Data Ingestion
`add()` → `resolve_data_directories` → `ingest_data` → `save_data_item_to_storage` → Create Dataset + Data records in relational DB

Key files: `cognee/api/v1/add/add.py`, `cognee/tasks/ingestion/ingest_data.py`

#### COGNIFY: Knowledge Graph Construction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [topoteretes/cognee](https://github.com/topoteretes/cognee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
