---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---


# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Insta RAG** is a modular, configuration-driven Python library for building advanced RAG (Retrieval-Augmented Generation) pipelines. It abstracts document processing, embedding, and hybrid retrieval behind a clean client, allowing developers to ship RAG applications faster.

Key features:
- **Semantic Chunking** – splits docs on topic boundaries to preserve context
- **Hybrid Retrieval** – semantic vectors + BM25 keyword search
- **HyDE Query Transform** – synthesizes hypothetical answers to improve recall
- **Reranking** – optional integration with SOTA rerankers (e.g., Cohere, Novita AI)
- **Pluggable by Design** – swap chunkers, embedders, rerankers, and vector DBs
- **Hybrid Storage** – Qdrant for vectors and MongoDB for flexible content storage
- **Graph RAG** *(NEW)* – Knowledge graph-based RAG using Neo4j and Graphiti for structured entity/relationship extraction

**Status**: Beta (v0.1.1-beta.3)
**Python**: 3.9+
**Package Manager**: uv (with fallback to pip)

### Graph RAG (Optional Feature)

Graph RAG is an optional feature that coexists with the existing vector-based RAG. Both systems can operate independently or complement each other.

**When to use Graph RAG:**
- Complex knowledge with many interconnected entities and relationships
- Need for explicit entity/relationship extraction and discovery
- Temporal awareness of facts and relationships
- Natural language understanding of "who", "what", "when" relationships

**Components:**
- `GraphRAGClient` – Independent async client for knowledge graph operations
- Neo4j backend – Graph database for storing entities and relationships
- Graphiti library – LLM-driven entity/relationship extraction
- No breaking changes to existing RAGClient API

**Usage pattern:**
```python
# Vector RAG (existing) - unchanged
rag_client = RAGClient(RAGConfig.from_env())

# Graph RAG (new) - separate, optional
graph_client = GraphRAGClient()
await graph_client.initialize()

# Both can coexist and be used independently or together
```

See `Graph RAG Architecture` section below for details.

## Development Setup

### Prerequisites
- Python 3.9 or higher
- Git
- uv (fast Python package manager): `curl -LsSf https://astral.sh/uv/install.sh | sh`

### Initial Setup
```bash
# Install the package in editable mode with dev dependencies
uv pip install -e . --group dev

# Install pre-commit hooks (includes ruff, commitizen, detect-secrets, etc.)
pre-commit install
pre-commit install --hook-type commit-msg
pre-commit install --hook-type pre-push
```

## Common Development Commands

### Code Quality
```bash
# Lint and auto-fix issues
ruff check . --fix

# Format code (Black-compatible style)
ruff format .

# Run all pre-commit hooks manually
pre-commit run --all-files

# Check for secrets in code
detect-secrets scan --baseline .secrets.baseline --only-verified
```

### Testing
```bash
# Run all tests
pytest

# Run tests with coverage
pytest --cov=src/insta_rag

# Run a specific test file
pytest tests/smoke_test.py

# Run tests matching a pattern
pytest -k "test_rag_client"

# Run with verbose output
pytest -v
```

### Version Management (using Commitizen)
```bash
# Bump a pre-release version (for beta releases)
cz bump --prerelease beta

# Bump patch version (0.1.0 -> 0.1.1)
cz bump --patch

# Bump minor version (0.1.0 -> 0.2.0)
cz bump --minor

# Bump major version (0.1.0 -> 1.0.0)
cz bump --major

# After bumping, push commits and tags
git push origin your-branch
git push origin --tags
```

## Project Structure

### Source Code (`src/insta_rag/`)

- **`core/`** – Main RAG client and configuration
  - `client.py` – `RAGClient` class, orchestrates all RAG operations (add, retrieve, update, delete documents)
  - `config.py` – Configuration management with dataclasses for VectorDB, Embedding, LLM, Chunking, etc.
  - `retrieval_method.py` – Enum for retrieval method selection (vector, keyword, hybrid)

- **`models/`** – Data models using Pydantic
  - `document.py` – `DocumentInput` for ingestion, supports text/PDF/URL sources
  - `chunk.py` – `Chunk` model for semantic units
  - `response.py` – Response models for API returns

- **`chunking/`** – Document splitting strategies
  - `semantic.py` – `SemanticChunker` uses topic boundaries for context preservation
  - Also includes fallback chunkers for various document types

- **`embedding/`** – Embedding providers
  - `openai.py` – OpenAI embeddings (text-embedding-3-large, etc.)
  - `base.py` – Abstract base for custom embedders
  - Support for Azure OpenAI and other providers via config

- **`retrieval/`** – Core retrieval logic
  - `vector_search.py` – Semantic search using vectors
  - `keyword_search.py` – BM25 keyword search
  - `query_generator.py` – HyDE query transformation (synthesizes hypothetical answers)
  - `reranker.py` – Reranking using cross-encoders (Cohere, Novita AI, etc.)
  - `base.py` – Base retriever interface

- **`vectordb/`** – Vector database integrations
  - `qdrant.py` – Qdrant vector store implementation
  - `base.py` – Abstract base for custom vector DBs

- **`utils/`** – Utility functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AI-Buddy-Catalyst-Labs/insta_rag](https://github.com/AI-Buddy-Catalyst-Labs/insta_rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
