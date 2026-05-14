---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an enterprise workshop series teaching the complete AI engineering lifecycle using LangChain, LangGraph, and LangSmith. It centers around building a customer support agent for TechHub, a fictional e-commerce store.

The workshop progresses through three modules:
1. **Agent Development** - Building multi-agent systems with human-in-the-loop
2. **Evaluation & Improvement** - Using offline evaluation to systematically improve agents
3. **Production Deployment** - Deploying to LangSmith with online evaluation and data flywheels

## Essential Commands

### Environment Setup
```bash
# Install dependencies (creates .venv automatically)
uv sync

# Configure environment
cp .env.example .env
# Edit .env with your API keys
# Optional: Set EMBEDDING_PROVIDER=openai if HuggingFace downloads are blocked

# Build vectorstore (required one-time setup, ~60 seconds)
# Uses HuggingFace embeddings by default (local, no API key)
# Set EMBEDDING_PROVIDER=openai in .env to use OpenAI embeddings instead
uv run python data/data_generation/build_vectorstore.py

# Note: Changing EMBEDDING_PROVIDER requires rebuilding the vectorstore
```

### Development
```bash
# Launch Jupyter for workshop notebooks
uv run jupyter lab

# Test LangGraph deployments locally
langgraph dev

# Test specific deployment graph
langgraph dev --graph supervisor_hitl_sql_agent
```

### Running Python Scripts
```bash
# All Python commands should use uv
uv run python <script.py>
```

## Architecture Overview

### Agent Factory Pattern

All agents use factory functions that return compiled LangGraph graphs:

```python
from agents import create_db_agent, create_docs_agent, create_supervisor_agent

# Development mode (with checkpointer for memory)
agent = create_db_agent(use_checkpointer=True)

# Production mode (LangSmith manages state)
agent = create_db_agent(use_checkpointer=False)
```

**Available Agents** (in `agents/`):
- `db_agent.py` - Rigid database tools for order/product queries
- `sql_agent.py` - Flexible SQL generation (improved version from Module 2)
- `docs_agent.py` - RAG search over product specs and policies
- `supervisor_agent.py` - Coordinates DB + Docs agents
- `supervisor_hitl_agent.py` - Full system with customer verification using LangGraph interrupts

### Multi-Agent System Architecture

The complete system (`supervisor_hitl_agent`) uses a three-stage flow:

1. **Classification** - Determines if query requires customer identity verification
2. **Verification (HITL)** - Uses `interrupt()` to collect and validate customer email
3. **Supervisor Routing** - Routes to specialized sub-agents (DB/SQL and Docs agents)

Key implementation details:
- Uses custom `IntermediateState(MessagesState)` to share `customer_id` between parent and subgraphs
- Subgraphs are added as nodes using `.add_node("supervisor", supervisor_graph)`
- Shared state keys (like `messages`) automatically flow between parent and subgraphs
- Dynamic prompts inject state (e.g., `customer_id`) at runtime

### State Management

**Development vs. Deployment:**
- **Local/Notebooks**: `use_checkpointer=True` with `MemorySaver()` for conversation memory
- **Production**: `use_checkpointer=False` because LangGraph Cloud provides managed persistence

**Custom State Schemas:**
```python
from langgraph.graph import MessagesState

class IntermediateState(MessagesState):
    customer_id: str  # Shared between parent and subgraphs
```

### Configuration System

Centralized in `config.py`:
- `DEFAULT_MODEL` - Workshop-wide model setting (override with `WORKSHOP_MODEL` env var)
- `DEFAULT_EMBEDDING_PROVIDER` - Embedding provider setting (override with `EMBEDDING_PROVIDER` env var, defaults to `huggingface`)
- `DEFAULT_DB_PATH` - Path to SQLite database
- `DEFAULT_VECTORSTORE_PATH` - Path to pre-built vectorstore (includes provider in filename)
- Path resolution handles both local dev and LangSmith deployment environments

All agents inherit settings from `config.py` but can override:
```python
agent = create_db_agent(model="anthropic:claude-sonnet-4")
```

## Key Files and Directories

### Core Modules
- `agents/` - Reusable agent factory functions
- `tools/` - Database tools (`database.py`) and RAG tools (`documents.py`)
- `evaluators/` - LLM-as-judge (`correctness_evaluator`) and trace-based metrics (`count_total_tool_calls_evaluator`)
- `deployments/` - Production-ready graph configurations referenced by `langgraph.json`

### Workshop Content
- `workshop_modules/` - 3 modules, 8 sections (Jupyter notebooks)
  - Work through sequentially: each builds on previous concepts
  - Start with `module_1/section_1_foundation.ipynb`

### Data
- `data/structured/techhub.db` - SQLite database (50 customers, 25 products, 250 orders)
- `data/documents/` - 30 markdown docs (product specs + policies) for RAG
- `data/vector_stores/techhub_vectorstore_{provider}.pkl` - Pre-built vectorstore (must run `build_vectorstore.py` first, provider is `huggingface` or `openai`)
- `data/structured/SCHEMA.md` - Complete database schema reference

### Deployment
- `langgraph.json` - Defines 6 deployable graphs for LangSmith

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langchain-ai/langsmith-agent-lifecycle-workshop](https://github.com/langchain-ai/langsmith-agent-lifecycle-workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
