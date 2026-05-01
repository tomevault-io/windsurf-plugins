---
trigger: always_on
description: > This file provides context for AI coding agents working with this codebase.
---

# AGENTS.md

> This file provides context for AI coding agents working with this codebase.

## Project Overview

**Agentic Local Brain** is a comprehensive personal knowledge management system designed to collect, process, and query knowledge from multiple sources. It features:

- **Multi-source Collection**: Files (PDF, Markdown, text), webpages, bookmarks, academic papers, emails, and notes
- **Intelligent Processing**: LLM-based tagging and vector embedding for semantic search
- **Flexible Retrieval**: Keyword search, semantic search, and RAG-based Q&A
- **Dual Interface**: CLI and REST API

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    User Interfaces                       │
│  ┌─────────────┐              ┌─────────────────────┐   │
│  │    CLI      │              │    Web API (REST)   │   │
│  │  (Click)    │              │     (FastAPI)       │   │
│  └──────┬──────┘              └──────────┬──────────┘   │
└─────────┼────────────────────────────────┼──────────────┘
          │                                │
          ▼                                ▼
┌─────────────────────────────────────────────────────────┐
│                    Core Modules                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐  │
│  │ Collectors  │  │ Processors  │  │     Query       │  │
│  │ - File      │  │ - Chunker   │  │ - Semantic      │  │
│  │ - Webpage   │  │ - Embedder  │  │ - Keyword       │  │
│  │ - Bookmark  │  │ - Tagger    │  │ - RAG           │  │
│  │ - Paper     │  │             │  │                 │  │
│  │ - Email     │  │             │  │                 │  │
│  │ - Note      │  │             │  │                 │  │
│  └─────────────┘  └─────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────┘
          │                                │
          ▼                                ▼
┌─────────────────────────────────────────────────────────┐
│                    Storage Layer                         │
│  ┌─────────────────────┐    ┌─────────────────────────┐ │
│  │   SQLite Storage    │    │    Chroma Storage       │ │
│  │   (Metadata, Tags)  │    │    (Vector Embeddings)  │ │
│  └─────────────────────┘    └─────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

## Directory Structure

```
kb/
├── __init__.py           # Package init with version
├── cli.py                # CLI entry point (Click framework)
├── config.py             # Configuration management (YAML-based)
├── collectors/           # Data collection modules
│   ├── base.py           # Abstract BaseCollector class
│   ├── file_collector.py # PDF, Markdown, text files
│   ├── webpage_collector.py
│   ├── bookmark_collector.py
│   ├── paper_collector.py
│   ├── email_collector.py
│   └── note_collector.py
├── processors/           # Content processing modules
│   ├── base.py           # Abstract BaseProcessor class
│   ├── chunker.py        # Document chunking
│   ├── embedder.py       # Text vectorization (DashScope/OpenAI)
│   ├── tag_extractor.py  # LLM-based tagging
│   └── wiki_compiler.py  # LLM-powered wiki article compilation (v0.7)
├── query/                # Search and retrieval
│   ├── models.py         # Data models (SearchResult, RAGResult, EnhancedRAGResult, etc.)
│   ├── semantic_search.py
│   ├── keyword_search.py
│   ├── rag.py            # RAG query implementation (v0.6)
│   ├── retrieval_pipeline.py  # Multi-stage retrieval orchestrator (v0.7)
│   ├── query_expander.py      # Query expansion and rewriting (v0.7)
│   ├── reranker.py            # LLM-based result reranking (v0.7)
│   ├── context_builder.py     # Token-aware context assembly (v0.7)
│   ├── conversation.py        # Multi-turn conversation management (v0.7)
│   ├── prompt_templates.py    # Configurable prompt templates (v0.7)
│   ├── graph_query.py         # Knowledge graph traversal (v0.6)
│   ├── topic_query.py         # Topic/cluster queries (v0.6)
│   └── reading_history.py     # Reading pattern tracking (v0.6)
├── storage/              # Data persistence
│   ├── sqlite_storage.py # Metadata and tags
│   └── chroma_storage.py # Vector storage
└── web/                  # REST API
    ├── app.py            # FastAPI application
    ├── dependencies.py   # Shared dependencies
    └── routes/           # API endpoints
        ├── dashboard.py
        ├── items.py
        ├── tags.py
        ├── search.py
        └── wiki.py        # Wiki article API endpoints (v0.7)
```

## Technology Stack

| Category | Technology | Version |
|----------|------------|---------|
| CLI Framework | Click | 8.0+ |
| Web Framework | FastAPI | 0.95+ |
| ASGI Server | Uvicorn | 0.20+ |
| Vector Storage | ChromaDB | 0.4+ |
| Metadata Storage | SQLite | Built-in |
| PDF Processing | PyPDF2 | 3.0+ |
| Web Scraping | httpx, readability-lxml | - |
| AI/ML (Embedding) | DashScope (text-embedding-v4) | 1.14+ |
| AI/ML (LLM) | DashScope (qwen-plus/qwen-max) | 1.14+ |
| LLM Integration | litellm | 1.30+ |
| Configuration | PyYAML | 6.0+ |
| Testing | pytest | 7.0+ |

## Entry Points

### CLI (`kb/cli.py`)

Main entry point configured in `pyproject.toml`:

```bash
# Installation creates 'kb' command

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent-creativity/agentic-local-brain](https://github.com/agent-creativity/agentic-local-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
