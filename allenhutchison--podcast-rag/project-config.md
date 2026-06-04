---
trigger: always_on
description: This document provides context for AI assistants working with this codebase.
---

# AGENTS.md - AI Assistant Context for Podcast RAG System

This document provides context for AI assistants working with this codebase.

## Project Overview

**Podcast RAG System** is a Python-based Retrieval-Augmented Generation (RAG) application that enables intelligent search and question-answering over podcast libraries.

**Core Functionality:**
- Transcribes audio using faster-whisper (CTranslate2-based Whisper)
- Extracts structured metadata using AI (Gemini)
- Stores transcriptions in vector databases for semantic search
- Answers natural language queries with source citations via CLI

**Tech Stack:** Python 3.11+, Gemini File Search, faster-whisper, Gemini API

## Architecture

### High-Level RAG Pipeline

```
User Query → Vector Search → Context Retrieval → Prompt Formatting → AI Model → Sourced Answer
```

### Key Components

1. **RAG Manager** (`src/rag.py`) - Orchestrates query processing pipeline
2. **Workflow System** (`src/workflow/`) - Pipeline-oriented processing for transcription → metadata → indexing
   - `orchestrator.py` - PipelineOrchestrator for continuous GPU-optimized processing
   - `workers/` - Individual workers for sync, download, transcription, metadata, indexing, cleanup
   - `post_processor.py` - Async post-processing thread pool
3. **Scheduler** (`src/scheduler.py`) - Entry point for running the pipeline
4. **Search & Storage:**
   - Gemini File Search - Semantic search with automatic chunking and embeddings
   - Database - Stores indexing status and metadata per episode
5. **MCP Server** (`src/mcp_server.py`) - Claude integration protocol

### Project Structure

```
/home/user/podcast-rag/
├── src/                    # Core application code
│   ├── rag.py              # RAG query orchestrator
│   ├── config.py           # Configuration management
│   ├── schemas.py          # Pydantic models for validation
│   ├── scheduler.py        # Pipeline entry point
│   │
│   ├── workflow/           # Processing pipeline
│   │   ├── orchestrator.py # PipelineOrchestrator
│   │   ├── config.py       # PipelineConfig
│   │   ├── post_processor.py # Async post-processing
│   │   └── workers/        # Individual stage workers
│   │
│   ├── db/
│   │   └── gemini_file_search.py  # Gemini File Search interface
│   │
│   ├── agents/             # Google ADK multi-agent system (web app)
│   │   ├── orchestrator.py # SequentialAgent + ParallelAgent setup
│   │   ├── podcast_search.py # PodcastSearchAgent with File Search
│   │   ├── web_search.py   # WebSearchAgent with google_search
│   │   └── synthesizer.py  # SynthesizerAgent for combining results
│   │
│   └── web/                # FastAPI web application
│       ├── app.py          # Main app with ADK integration
│       └── static/         # Frontend (Tailwind CSS + vanilla JS)
│
├── scripts/
│   └── file_search_utils.py      # File Search management utilities
│
├── prompts/                # AI prompt templates
│   └── metadata_extraction.txt   # Metadata extraction
│
├── docs/                   # Documentation
│   ├── docker.md           # Docker deployment guide
│   ├── web-app.md          # Web application guide
│   ├── deploy-vps.md       # VPS deployment + cutover runbook (current)
│   └── faster-whisper-benchmark.md  # Benchmark analysis
│
├── tests/                  # pytest test suite
├── pyproject.toml          # Project config and dependencies (uv)
└── uv.lock                 # Locked dependencies
```

## Key Files Reference

| File | Purpose | When to Modify |
|------|---------|----------------|
| `src/config.py` | Environment variables, paths, settings | Adding new config options |
| `src/rag.py` | Query processing, AI inference | Changing RAG logic |
| `src/db/gemini_file_search.py` | Gemini File Search interface | File Search integration changes |
| `src/workflow/orchestrator.py` | PipelineOrchestrator for processing | Processing workflow changes |
| `src/workflow/workers/` | Individual processing stage workers | Adding/modifying processing stages |
| `src/gemini_search.py` | Search manager using Gemini File Search | Search logic modifications |
| `src/agents/` | Google ADK multi-agent system | Web app agent behavior |
| `src/web/app.py` | FastAPI web application | Web interface changes |
| `prompts/metadata_extraction.txt` | Metadata extraction prompt | Improving metadata quality |
| `pyproject.toml` | Python dependencies (uv) | Adding/updating packages |

## Development Workflow

### Setup

This project uses [uv](https://docs.astral.sh/uv/) for dependency management.

```bash
# 1. Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh
# Or on macOS: brew install uv

# 2. Install dependencies and create virtual environment
uv sync

# 3. Activate virtual environment (optional if using 'uv run')
source .venv/bin/activate

# 4. Configure Doppler for secrets management
# Ensure you have access to the project's Doppler config

# 5. Initialize/migrate the database (use doppler for env vars)
doppler run -- alembic upgrade head

# 6. System dependencies
# Requires: ffmpeg
```

### Common Tasks

This project uses [poethepoet](https://poethepoet.naez.io/) for task running. **Use `doppler run --` prefix** for commands that need environment variables.

| Command | Description |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [allenhutchison/podcast-rag](https://github.com/allenhutchison/podcast-rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
