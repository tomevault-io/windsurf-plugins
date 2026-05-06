---
trigger: always_on
description: Provides read-only tools for AI assistants:
---

# Legal MCP - Architecture and Development Guide

## Project Overview

Legal MCP is a semantic search system for German legal texts with two main components:

1. **Store API** (FastAPI) - Backend service with PostgreSQL + pgvector for storing and searching legal texts
2. **MCP Server** (FastMCP) - Model Context Protocol server that exposes legal search tools to AI assistants

**Purpose**: Enable AI assistants to semantically search German legal codes (BGB, StGB, GG, etc.) using natural language queries and retrieve specific legal sections.

## Architecture

```
┌─────────────────┐
│   AI Assistant  │
│  (via MCP SDK)  │
└────────┬────────┘
         │ stdio/http
         ▼
┌─────────────────┐
│   MCP Server    │  (Port 8001, FastMCP)
│                 │  Provides tools for searching legal texts
└────────┬────────┘
         │
         │ HTTP REST calls
         ▼
┌─────────────────┐
│   Store API     │  (Port 8000, FastAPI)
│                 │  REST endpoints for import, query, search
└────────┬────────┘
         │
         │ SQLAlchemy ORM (asyncpg)
         ▼
┌─────────────────┐
│   PostgreSQL    │  (Port 5432)
│   + pgvector    │  Stores legal texts with embeddings
└────────┬────────┘
         │
         │ HTTP API calls
         ▼
┌─────────────────┐
│     Ollama      │  (Port 11434, runs on host)
│                 │  Generates embeddings for semantic search
└─────────────────┘
```

## Directory Structure

```
legal-mcp/
├── store/                          # Store API (FastAPI backend)
│   ├── app/
│   │   ├── main.py                 # FastAPI app initialization
│   │   ├── config.py               # Settings (Pydantic Settings)
│   │   ├── database.py             # SQLAlchemy setup (async + sync engines)
│   │   ├── models.py               # Pydantic + SQLAlchemy models
│   │   ├── repository.py           # Data access layer
│   │   ├── embedding.py            # Ollama integration
│   │   ├── dependencies.py         # FastAPI dependencies
│   │   ├── routers/                # API route handlers
│   │   └── scrapers/               # Legal text scrapers
│   │       └── gesetze_im_internet/
│   ├── alembic/                    # Database migrations
│   ├── tests/                      # Test suite
│   └── Dockerfile                  # Store API container
│
├── mcp/                            # MCP Server (FastMCP)
│   ├── server/
│   │   └── main.py                 # MCP server with tools
│   └── Dockerfile                  # MCP server container
│
├── docker-compose.yml              # Orchestration
├── requirements.txt                # Python dependencies
├── Makefile                        # Development commands
└── README.md                       # User documentation
```

## Key Components

### Store API (FastAPI)

Located in `store/app/`:

- **main.py** - FastAPI application entry point
- **models.py** - Data models (Pydantic for API, SQLAlchemy for database)
- **repository.py** - Database operations abstraction layer
- **embedding.py** - Embedding generation via Ollama
- **routers/legal_texts.py** - REST endpoints for legal texts
- **scrapers/** - Downloads and parses legal texts from sources

### MCP Server (FastMCP)

Located in `mcp/server/main.py`:

Provides read-only tools for AI assistants:
- `search_legal_texts` - Semantic search
- `get_legal_section` - Retrieve specific sections
- `get_available_codes` - List all available legal codes in the database

### Database

PostgreSQL with pgvector extension for vector similarity search. The main table stores:
- Legal text content
- Vector embeddings (2560-dimensional)
- Metadata (code, section, sub-section)
- Unique constraint on (code, section, sub_section) for upserts

Migrations managed via Alembic in `store/alembic/versions/`.

### Scrapers

Located in `store/app/scrapers/gesetze_im_internet/`:

- Downloads XML from gesetze-im-internet.de
- Parses gii-norm.dtd format
- Extracts structured legal text sections
- All scrapers implement the `Scraper` abstract base class

## Development Workflow

### Quick Start

```bash
# Start all services
make up

# Run migrations
make migrate

# Import a legal code
curl -X POST http://localhost:8000/legal-texts/gesetze-im-internet/bgb
```

### Common Commands

```bash
make help           # Show all available commands
make build          # Build Docker containers
make up             # Start services
make down           # Stop services
make logs           # View logs
make clean          # Remove containers + volumes
make migrate        # Run database migrations
make test           # Run tests
make shell-store    # Shell into store-api container
make shell-db       # PostgreSQL shell
```

See `Makefile` for the complete list of available commands.

### Local Development

```bash
# Start only PostgreSQL
docker-compose up postgres -d

# Set up Python environment
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# Run migrations
cd store && alembic upgrade head

# Start Store API
cd store && uvicorn app.main:app --reload

# Start MCP Server
cd mcp && python -m server.main
```

## Configuration

Environment variables are managed via Pydantic Settings in `store/app/config.py`.

Key variables:
- Database connection (POSTGRES_HOST, POSTGRES_PORT, etc.)
- Ollama endpoint (OLLAMA_BASE_URL)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ayunis-core/ayunis-legal-mcp](https://github.com/ayunis-core/ayunis-legal-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
