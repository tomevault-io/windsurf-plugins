---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RAGFlow is an open-source RAG (Retrieval-Augmented Generation) engine. This repository contains "TaskMe" - an enhanced version based on RAGFlow v0.17 that focuses on knowledge base management and intelligent Agent functionality, with traditional chat features removed.

## Architecture

### Backend Structure
- **Framework**: Python Flask + Gunicorn
- **API Apps**: Located in `/api/apps/` with dynamic Blueprint registration
- **Core Engine**: RAG engine in `/rag/` directory
- **Services**: Database services in `/api/db/services/`

### Frontend Structure  
- **Framework**: React 18 + UmiJS 4 + Ant Design 5
- **State Management**: Zustand + React Query
- **Styling**: Less + TailwindCSS
- **Key Pages**: `/web/src/pages/` (knowledge, agent, file-manager, etc.)

### Database Architecture
- **MySQL**: Structured data (users, knowledge bases, conversations)
- **Elasticsearch/Infinity**: Document vector search
- **Redis**: Session management and caching
- **MinIO**: File storage

## Development Commands

### Server Startup
```bash
# Development mode (requires virtual environment)
source .venv/bin/activate
export PYTHONPATH=$(pwd)
./start_server.sh

# Production mode
gunicorn -c gunicorn.conf.py "api.ragflow_server:create_app()"

# Docker deployment
cd docker/
docker compose up -d
```

### Frontend Development
```bash
cd web/
npm install
npm run dev      # Development server
npm run build    # Production build
npm test         # Run tests
```

### Testing and Validation
```bash
# Python tests
pytest

# Test server startup (critical for validating changes)
source .venv/bin/activate && ./start_server.sh
```

## Critical Development Notes

### Development Server Status
**NOTE**: The development servers (both backend and frontend) are currently running in reload mode. Code changes are automatically reloaded without requiring manual server restart. Testing can be done immediately after making changes.

### Server Startup Testing
**IMPORTANT**: When servers are not running, always test server startup after making changes to the codebase. Many import errors and configuration issues only surface during server initialization. Use `./start_server.sh` to validate that all modules load correctly.

### Proxy Configuration
The startup script includes proxy cleanup to avoid SOCKS proxy issues that can cause import failures with httpx/ollama dependencies.

### API App Registration
- Apps in `/api/apps/` are dynamically loaded by `/api/apps/__init__.py`
- Each app file gets a `manager` Blueprint assigned at runtime via `register_page()`
- Never define `manager = None` in app files - this causes import errors
- Use `@manager.route()` decorators with `# noqa: F821` to suppress linting

### Table Knowledge Base Feature
New functionality for database/Excel knowledge bases:
- **Backend**: `/api/apps/table_kb_app.py`, `/rag/table_knowledge/`
- **Frontend**: `/web/src/pages/table-knowledge-base/`, `/web/src/components/table-knowledge-base/`
- **API Routes**: `/v1/kb/table/*` endpoints

### Configuration Files
- **Main Config**: `/conf/service_conf.yaml` (from template)
- **Frontend Config**: `/web/src/conf.json`
- **Environment**: Docker `.env` files
- **Dependencies**: `pyproject.toml` for Python, `package.json` for Node.js

## Project Structure Highlights

### Key Directories
```
/api/apps/           # Flask Blueprint applications
/rag/               # Core RAG engine and processing
/web/src/pages/     # React page components  
/web/src/services/  # API service layer
/conf/              # Configuration templates
/docker/            # Container deployment files
```

### Database Models
Located in `/api/db/db_models.py` with migration support. Notable additions include:
- `storage_type` and `table_config` fields in Knowledgebase model
- `TableKnowledgeBaseMetadata` model for enhanced table KB context

### Service Layer
Database services in `/api/db/services/` follow consistent patterns:
- Permission checks via `accessible()` and `accessible4deletion()`
- Error handling with `get_data_error_result()` and `server_error_response()`
- Standardized JSON responses via `get_json_result()`

## Development Environment

### Requirements
- **Python**: 3.10-3.12
- **Node.js**: >=18.20.4  
- **System**: CPU>=4 cores, RAM>=16GB, Disk>=50GB
- **Docker**: >=24.0.0 for container deployment

### Virtual Environment
The project uses `.venv/` for Python virtual environment. Always activate before development and ensure `PYTHONPATH` is set to the project root.

## Frontend-Backend Interaction Architecture

### API Design Patterns

#### 1. URL Structure and Routing
RAGFlow follows RESTful API design principles with consistent URL patterns:

```
Base URL: /v1/{module}/{action}
Examples:
- /v1/kb/list              # Knowledge base list
- /v1/kb/create            # Create knowledge base  
- /v1/kb/table/query       # Table KB query
- /v1/user/info            # User information
- /v1/llm/factories        # LLM factory list
```

**URL Pattern Rules:**
- Always prefixed with `/v1` for API versioning
- Module-based organization (`kb`, `user`, `llm`, `flow`, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zalan159/taskme](https://github.com/zalan159/taskme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
