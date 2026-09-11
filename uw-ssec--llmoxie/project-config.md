---
trigger: always_on
description: > Essential context for AI coding assistants. See `agent_docs/` for detailed
---

# AGENTS.md - LLMaven AI Assistant Guide

> Essential context for AI coding assistants. See `agent_docs/` for detailed
> guides.

---

## Project Overview

**LLMaven** is a scientific research tool that extends LLMs with domain-specific
knowledge using Retrieval Augmented Generation (RAG).

**Users**: Astrophysics researchers working with Rubin Observatory/LSST data.

**Purpose**: Enable researchers to query scientific literature and datasets
using natural language, with answers grounded in domain-specific sources.

**Stack**: Python package with FastAPI backend, Streamlit frontend, and Azure
deployment via Pulumi.

---

## Directory Map

| Path                          | Purpose                  | Notes                             |
| ----------------------------- | ------------------------ | --------------------------------- |
| `src/llmaven/`                | Main installable package | Core development                  |
| `src/llmaven/v1/`             | REST API v1 endpoints    | Route handlers                    |
| `src/llmaven/core/`           | ML/AI components         | Embeddings, retrieval, generation |
| `src/llmaven/services/`       | Business logic           | Service orchestration             |
| `src/llmaven/schemas/`        | Pydantic models          | API contracts                     |
| `src/llmaven/frontend/`       | Streamlit UI             | User interface                    |
| `src/llmaven/agentic/`        | Agentic RAG system       | Ingestion, agents, vector store   |
| `src/llmaven/infrastructure/` | Pulumi resources         | Azure deployment                  |
| `archive/`                    | Archived code            | **DO NOT MODIFY**                 |
| `tests/`                      | Test suite               | pytest                            |

---

## Essential Commands

```bash
# Environment
pixi install                              # Install dependencies
pixi shell -e llmaven                     # Enter environment

# Development
llmaven server serve --env development --reload  # API (localhost:8000)
llmaven server ui                                # Streamlit (localhost:8501)

# Agentic RAG (NEW)
llmaven agentic ingest ./docs             # Ingest documents
llmaven agentic search "query"            # Hybrid search
llmaven agentic chat                      # Interactive RAG chat

# Testing & Validation
pytest --cov=llmaven                      # Run tests with coverage
pre-commit run --all-files                # Lint and format
```

---

## Key Technologies

| Category        | Technology              | Purpose                                    |
| --------------- | ----------------------- | ------------------------------------------ |
| API             | FastAPI                 | REST endpoints                             |
| UI              | Streamlit               | Interactive frontend                       |
| Vector DB       | Qdrant                  | Semantic search                            |
| LLM             | LangChain + HuggingFace | RAG orchestration                          |
| Agentic RAG     | pydantic-ai + fastembed | Hybrid search with multi-vector embeddings |
| Infra           | Pulumi                  | Azure deployment                           |
| Package Manager | Pixi                    | Dependencies                               |

---

## Documentation Index

Before starting work, review relevant docs in `agent_docs/`:

| Document                                                | When to Read                     |
| ------------------------------------------------------- | -------------------------------- |
| [`adding_endpoints.md`](agent_docs/adding_endpoints.md) | Adding new API endpoints         |
| [`code_conventions.md`](agent_docs/code_conventions.md) | Naming patterns, style questions |
| [`commit_messages.md`](agent_docs/commit_messages.md)   | Writing commit messages          |
| [`infrastructure.md`](agent_docs/infrastructure.md)     | Pulumi/Azure deployment          |
| [`troubleshooting.md`](agent_docs/troubleshooting.md)   | Debugging common issues          |

---

## Critical Reminders

- **Never modify `archive/`** — Contains legacy code for reference only
- **Run `pre-commit run --all-files`** before committing
- **Configuration file `llmaven-config.yaml`** is gitignored (contains secrets)

---

## Agentic RAG System

The Agentic RAG system is a next-generation retrieval and question-answering
system that combines hybrid search (Dense + Sparse + ColBERT) with intelligent
agent-based answer generation. It provides superior retrieval accuracy compared
to the legacy single-vector search system.

### Architecture Overview

The agentic RAG system consists of four main components:

1. **Ingestion Pipeline** (`src/llmaven/agentic/ingestion/pipeline.py`)
   - Multi-format document processing with `docling`
   - Intelligent chunking that preserves document structure
   - Multi-vector embedding generation (Dense, Sparse, ColBERT)
   - Batch processing with progress indicators

2. **Vector Store** (`src/llmaven/agentic/vector_store/qdrant_manager.py`)
   - Qdrant Named Vectors support
   - Collection management and validation
   - Hybrid search operations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uw-ssec/llmoxie](https://github.com/uw-ssec/llmoxie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
