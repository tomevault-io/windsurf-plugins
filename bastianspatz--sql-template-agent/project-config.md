---
trigger: always_on
description: A FastAPI backend for automotive trace analysis that enables engineers to analyze vehicle trace data through natural language queries. The system uses a ReAct agent that autonomously picks templates or generates SQL to answer queries.
---

# Trace Analysis API - Project Context

## Project Overview

A FastAPI backend for automotive trace analysis that enables engineers to analyze vehicle trace data through natural language queries. The system uses a ReAct agent that autonomously picks templates or generates SQL to answer queries.

**Core Workflow:** Ticket ID → Generate Data → Ask Agent Questions

---

## Architecture Principles

### 1. **Simple > Complex**
- Flat file structure, no unnecessary nesting
- Each file has ONE responsibility
- No premature abstractions

### 2. **Agent-First**
- Single entry point: the ReAct agent (`POST /analyze`)
- Agent autonomously searches templates or writes SQL
- Template search and execution kept as secondary endpoints

### 3. **Extensible by Design**
- Add new templates without touching core code
- Templates are self-contained modules

### 4. **Type-Safe**
- Pydantic models everywhere
- Request/response validation automatic
- Clear contracts between components

---

## High-Level Architecture

```
┌─────────────────────────────────────────────┐
│  FastAPI REST API                            │
│                                              │
│  Primary:                                    │
│  • /analyze          - Agent analysis       │
│  • /generate-data    - Sample data gen      │
│  • /schema           - DB schema info       │
│  • /tables/*         - Table browsing       │
│                                              │
│  Secondary (kept for later):                 │
│  • /discover         - Find templates       │
└──────────────┬──────────────────────────────┘
               │
┌──────────────┴──────────────────────────────┐
│  Agent (LangGraph ReAct)                     │
│  3 tools: search_templates,                  │
│           execute_template, execute_sql      │
└──────────────┬──────────────────────────────┘
               │
┌──────────────┴──────────────────────────────┐
│  Data & Integration Layer                    │
│  • DataManager       - PostgreSQL operations│
│  • TemplateSearch    - Weaviate hybrid search│
│  • LLMClient         - OpenAI/Anthropic/Ollama│
│  • JiraClient        - Ticket context       │
└──────────────────────────────────────────────┘
```

---

## Technology Stack

### Core Framework
- **FastAPI** (0.115+) - Modern async web framework
- **Pydantic** (2.9+) - Data validation and settings
- **Uvicorn** - ASGI server

### Data & SQL
- **PostgreSQL** - Analytical SQL database (per-ticket schema isolation)
- **psycopg** (3.x) - PostgreSQL driver with connection pooling
- **Pandas** (2.2+) - Data manipulation (cursor-based fetching, no `pd.read_sql`)

### LLM & AI
- **LangChain** - LLM abstraction (OpenAI, Anthropic, Ollama)
- **LangGraph** - ReAct agent loop

### Search
- **Weaviate** - Hybrid vector + keyword search for template discovery
- **Ollama** - Local embeddings (`nomic-embed-text`)

### Frontend
- **Next.js 16** + **React 19** + **TailwindCSS**
- **Axios** for API calls
- **react-markdown** for rendering agent responses (with prose CSS styling for code blocks, tables, blockquotes, lists)
- Sidebar + chat layout (not tabs) — ticket list in sidebar, chat in main area
- Inline tool results show SQL, data tables, template search cards, and source table name chips extracted from SQL

### Development
- **uv** - Fast Python package manager
- **Pytest** - Testing
- **Black** - Code formatting
- **Ruff** - Fast linting

---

## Project Structure

```
backend/
└── app/
    ├── main.py              # FastAPI app, lifespan, lazy singletons
    ├── config.py            # Settings (env-based)
    ├── routes.py            # All API routes
    ├── agent.py             # LangGraph ReAct agent, tools, prompt assembly
    ├── llm.py               # LLM client (OpenAI/Anthropic/Ollama)
    ├── database.py          # PostgreSQL operations (schema isolation)
    ├── search.py            # Weaviate hybrid search
    ├── jira.py              # Jira ticket context (in-memory)
    ├── models.py            # All request/response Pydantic models
    ├── data.py              # Sample data generators
    ├── skills/              # Agent skill files loaded into system prompt
    │   ├── template_analysis.md
    │   ├── freehand_sql.md
    │   └── iterative_analysis.md
    └── templates/           # SQL template system
        ├── base.py          # SQLTemplate model
        ├── library.py       # Template registry
        └── registry/        # Individual templates
            ├── error_lookup.py
            ├── time_window.py
            ├── count_occurrences.py
            ├── someip_errors.py
            ├── voltage_anomaly.py
            └── ecu_communication.py

react_frontend/
└── app/
    ├── page.tsx                    # Root page (sidebar + chat layout)
    ├── layout.tsx                  # Next.js layout
    ├── globals.css                 # Theme variables, prose styling, animations
    ├── lib/
    │   ├── api.ts                  # Backend API client
    │   ├── attachments.ts          # Attachment workflow helpers
    │   └── parseToolOutput.ts      # Tool result parser + SQL table extraction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BastianSpatz/sql_template_agent](https://github.com/BastianSpatz/sql_template_agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
