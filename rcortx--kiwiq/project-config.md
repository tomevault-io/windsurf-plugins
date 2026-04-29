---
trigger: always_on
description: IMPORTANT: Once you set yourself some todos, you should complete them without stopping,
---

# CLAUDE.md

IMPORTANT: Once you set yourself some todos, you should complete them without stopping, 
*unless*
 you have some particular 
*reason*
 to stop (e.g. you need to ask a question, you need feedback / input, etc.) If you do stop before finishing all todos, you MUST state the reason why you're stopping. We'll call this the "no-stop" rule. If you stop before finishing todos, and don't give any reason why you've stopped, I'll be very disappointed, and I'll ask you why you didn't follow the "no-stop rule."
Never be lazy and ask me obvious questions, just to stop in between completing your TODOs!


DON"T USE SED for EDITING!!

Never duplicate, redundant code, keep it modular and reuse it, maybe create shared helper utils which is better maintainable in the long term.

## Project Overview

KiwiQ AI Platform - A multi-service, multi-tier Kubernetes-based platform for AI-powered workflow orchestration with LangGraph, Prefect, and comprehensive integrations for LLM providers, scraping, and customer data management.

**Tech Stack:**
- Python 3.12 (dictated by Airflow compatibility)
- Poetry for dependency management
- FastAPI for REST APIs
- Prefect for workflow orchestration
- LangGraph for workflow graphs
- PostgreSQL (via SQLModel/SQLAlchemy/Alembic)
- MongoDB (customer data, versioned documents)
- Redis (caching, queuing)
- RabbitMQ (event streaming)
- Weaviate (vector database)
- Docker Compose for local development

## Repository Structure

```
kiwiq-backend/
├── libs/src/              # Shared cross-service libraries
│   ├── db/               # Database clients, models, migrations (Alembic)
│   ├── global_config/    # Global settings
│   ├── global_utils/     # Shared utilities
│   ├── mongo_client/     # MongoDB client
│   ├── rabbitmq_client/  # RabbitMQ client
│   ├── redis_client/     # Redis client
│   └── weaviate_client/  # Weaviate vector DB client
├── services/
│   ├── kiwi_app/                    # Core FastAPI application
│   │   ├── auth/                    # Authentication & authorization
│   │   ├── billing/                 # Stripe billing & credits
│   │   ├── workflow_app/            # Workflow backend APIs
│   │   │   ├── routes.py           # Workflow execution endpoints
│   │   │   ├── websockets.py       # Real-time workflow updates
│   │   │   ├── event_consumer.py   # RabbitMQ event processing
│   │   │   └── app_state.py        # Application state management
│   │   ├── data_jobs/              # Data ingestion & RAG
│   │   ├── rag_service/            # RAG query endpoints
│   │   └── main.py                 # FastAPI app entrypoint
│   ├── workflow_service/           # Core workflow engine
│   │   ├── registry/               # Node & workflow registry
│   │   │   ├── nodes/             # All node implementations
│   │   │   │   ├── core/          # Core nodes (router, map_list_router, etc.)
│   │   │   │   ├── llm/           # LLM nodes with tool support
│   │   │   │   ├── data_ops/      # Transform, merge, aggregate
│   │   │   │   ├── db/            # Customer data CRUD nodes
│   │   │   │   ├── scraping/      # LinkedIn & web scraping nodes
│   │   │   │   └── tools/         # Document CRUD tools
│   │   │   ├── schemas/           # Base schemas & reducers
│   │   │   └── workflows/         # Workflow definitions
│   │   ├── graph/                 # Graph builder & runtime
│   │   │   ├── builder.py         # GraphSchema -> LangGraph
│   │   │   ├── graph.py           # GraphSchema definitions
│   │   │   └── runtime/           # LangGraph runtime adapter
│   │   └── services/
│   │       ├── worker.py          # Prefect worker entrypoint
│   │       ├── scraping/          # Web scraping service
│   │       └── external_context_manager.py
│   ├── linkedin_integration/      # LinkedIn OAuth & API
│   └── scraper_service/           # Scraping API routes
├── standalone_test_client/        # Workflow testing client
│   └── kiwi_client/              # Test workflows & utilities
├── tests/
│   ├── unit/                     # Unit tests
│   └── integration/              # Integration tests
├── docs/design_docs/             # Architecture documentation
├── docker/                       # Docker configurations
├── pyproject.toml               # Poetry dependencies
└── pytest.ini                   # Test configuration
```

## Development Setup

### Environment Setup

1. **Install dependencies:**
   ```bash
   poetry install
   ```

2. **Configure environment:**
   - Copy `.env.sample` to `.env`
   - Set required values for databases, API keys, etc.

3. **Start services (development):**
   ```bash
   docker compose -f docker-compose-dev.yml up --build
   ```

4. **Access API documentation:**
   - Local: http://localhost:8000/docs
   - Production: Configure via API_BASE_HOST env var

### Python Path Configuration

The project uses a custom Python path setup for importing local libraries and services:

```bash
PYTHONPATH=$(pwd):$(pwd)/services
```

This is already configured in `.env` for IDE support. VS Code/Cursor automatically loads this.

### Import Conventions

- **Shared libraries:** Import from `libs/src` modules directly
  ```python
  from db.session import get_async_session
  from global_config.settings import global_settings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rcortx/kiwiq](https://github.com/rcortx/kiwiq) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
