---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Arthur Engine is a Python-based AI/ML monitoring and governance platform with three main components:

- **GenAI Engine**: FastAPI-based REST API for LLM evaluation and guardrailing
- **ML Engine**: Job-based evaluation engine for ML model monitoring
- **Frontend UI**: React + TypeScript + Vite web application

## Technologies

**Backend:**

- Python 3.12 (GenAI Engine), Python 3.13 (ML Engine)
- FastAPI, SQLAlchemy, PostgreSQL with pgVector
- OpenAI/Azure LLMs, LangChain, LiteLLM
- ML Models: Transformers, Sentence Transformers, Spacy
- NER/PII: Presidio, GLiNER
- Alembic for database migrations

**Frontend:**

- React 19, TypeScript, Vite
- **MUI (Material UI) v7** - Primary component library (`@mui/material`, `@mui/icons-material`, `@mui/x-date-pickers`)
- Emotion (`@emotion/react`, `@emotion/styled`) - Styling engine for MUI
- Tailwind CSS - Utility classes for layout supplementing MUI
- TanStack Query/Table, Material React Table
- Zustand for state management

**Infrastructure:**

- Docker, Docker Compose, Helm, AWS ECS
- OpenTelemetry, NewRelic
- Pytest, Coverage, Locust

## Common Commands

### GenAI Engine

```bash
# Setup
cd genai-engine
uv sync --group dev --group linters

# Start PostgreSQL (required)
docker compose up

# Database setup
export POSTGRES_USER=postgres
export POSTGRES_PASSWORD=changeme_pg_password
export POSTGRES_URL=localhost
export POSTGRES_PORT=5432
export POSTGRES_DB=arthur_genai_engine
export GENAI_ENGINE_SECRET_STORE_KEY="some_test_key"
uv run alembic upgrade head

# Run development server
export PYTHONPATH="src:$PYTHONPATH"
uv run serve
# Access at http://localhost:3030/docs

# Testing
uv run pytest -m "unit_tests"
uv run pytest -m "unit_tests" --cov=src --cov-fail-under=79
./tests/test_remote.sh  # Integration tests

# Database migrations
uv run alembic revision --autogenerate -m "<message>"
uv run alembic upgrade head

# Code quality
uv run isort src --profile black
uv run autoflake --remove-all-unused-imports --in-place --recursive src
uv run black src
uv run routes_security_check

# Generate API changelog
uv run generate_changelog
```

### ML Engine

```bash
cd ml-engine

# Generate GenAI Engine client
cd scripts
./openapi_client_utils.sh generate python
./openapi_client_utils.sh install python
./install_db_dependencies.sh
cd ..

uv sync

# Run ML Engine
uv run python src/ml_engine/job_agent.py

# Testing
uv sync --group dev
uv run pytest tests/unit

# Code quality
uv run isort src/ml_engine --profile black --check
uv run black --check src/ml_engine
uv run mypy src/ml_engine
```

### Frontend UI

```bash
cd genai-engine/ui
yarn install
yarn dev              # Development at localhost:5173
yarn build           # Production build
yarn type-check      # TypeScript checking
yarn lint            # ESLint
yarn format          # Prettier (auto-fix)
yarn format:check    # Prettier (check only)
yarn generate-api    # Generate API client from OpenAPI spec

# Before committing (REQUIRED - CI enforced)
yarn check           # Runs type-check, lint, and format:check
```

### Docker Compose (Full Stack)

```bash
cd deployment/docker-compose/genai-engine
cp .env.template .env
# Edit .env with your configuration
docker compose up
# Access at http://localhost:3030/docs
```

## Architecture

### GenAI Engine Structure

```
src/
├── server.py              # FastAPI app initialization
├── dependencies.py        # Dependency injection (DB, auth, clients)
├── config/                # Configuration management
├── auth/                  # Authentication & OAuth (Keycloak, JWT)
├── db_models/             # SQLAlchemy models (19 entity types)
│   ├── task_models.py            # Task/use-case definitions
│   ├── rule_models.py            # Rule configurations
│   ├── rule_result_models.py     # Rule evaluation results
│   ├── inference_models.py       # Span/trace data storage
│   └── dataset_models.py         # Dataset management
├── repositories/          # Data access layer (24 repositories)
│   ├── tasks_repository.py
│   ├── rules_repository.py
│   ├── inference_repository.py
│   └── span_repository.py        # Trace data queries
├── routers/               # API route handlers
│   ├── v1/                # Legacy API endpoints
│   │   ├── trace_api_routes.py
│   │   ├── llm_eval_routes.py
│   │   └── rag_routes.py
│   └── v2/                # Current API version
│       ├── task_management_routes.py
│       ├── rule_management_routes.py
│       ├── validate_routes.py
│       └── feedback_routes.py
├── scorer/                # Evaluation engine
│   ├── scorer.py          # Main scorer orchestration
│   ├── llm_client.py      # OpenAI/Azure/LiteLLM integration
│   └── checks/            # Evaluation implementations
│       ├── hallucination/         # Claim-based LLM judge
│       ├── prompt_injection/      # DebertaV3 model
│       ├── toxicity/              # RoBERTa classifier
│       ├── pii/                   # Presidio + GLiNER
│       ├── sensitive_data/        # Few-shot LLM judge
│       └── regex/                 # Pattern-based checks
├── schemas/               # Pydantic request/response models

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arthur-ai/arthur-engine](https://github.com/arthur-ai/arthur-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
