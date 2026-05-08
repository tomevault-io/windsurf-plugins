---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Agent Studio Dispatch is a message forwarding service that establishes a communication channel between WeChat Work (Enterprise WeChat) bots and external Agent Studio services. It acts as a bridge, receiving messages from WeChat Work bots, forwarding them to configured endpoints, and returning responses to users.

**Core functionality:**
- Bidirectional message forwarding between WeChat Work and Agent Studio
- Multi-bot management with individual configurations
- WebSocket tunnel support for internal network penetration (via tunely)
- Access control (whitelist/blacklist) per bot
- User project management for flexible routing
- Session persistence for conversation continuity
- Admin slash commands for bot/project/tunnel management

## Development Commands

### Running the Application

```bash
# Standard startup (database mode, recommended)
USE_DATABASE=true uv run python -m forward_service.app

# With specific port
FORWARD_PORT=8084 uv run python -m forward_service.app

# With MySQL database
DATABASE_URL="mysql+pymysql://user:pass@host:port/db" uv run python -m forward_service.app

# Development mode with SQLite
DATABASE_URL="sqlite+aiosqlite:///./data/forward_service.db" uv run python -m forward_service.app
```

### Database Operations

```bash
# Run database migrations (required after schema changes)
alembic upgrade head

# Create a new migration after modifying models.py
alembic revision --autogenerate -m "description of changes"

# View current migration version
alembic current

# Rollback one migration
alembic downgrade -1

# Generate SQL for production review (offline mode)
alembic upgrade head --sql
```

### Testing

```bash
# Run all tests
uv run pytest

# Run specific test file
uv run pytest tests/test_callback.py

# Run with coverage report
uv run pytest --cov=forward_service --cov-report=html

# Run end-to-end tunnel tests
uv run pytest tests/test_e2e_tunnel.py -v

# Run a single test function
uv run pytest tests/test_callback.py::test_callback_handler -v
```

### Linting and Code Quality

```bash
# Run ruff linter
uv run ruff check forward_service tests

# Auto-fix issues
uv run ruff check --fix forward_service tests

# Note: E712 is ignored for SQLAlchemy (requires `== True/False` for SQL expressions)
```

## Architecture Overview

### Application Flow

```
WeChat Work Bot → POST /callback → Forward Service → Agent Studio API
                      ↓                                    ↓
                 Access Control                         Response
                      ↓                                    ↓
              Session Lookup                    Message Splitter
                      ↓                                    ↓
                 Forward Logic → Log → Reply to WeChat Work
```

### Key Components

1. **Webhook Handler** (`/callback` endpoint)
   - Receives messages from WeChat Work bots
   - Supports multi-bot configuration via URL parameters
   - Entry point: `forward_service/routes/callback.py`

2. **Configuration Manager** (`config.py`)
   - Database-first configuration (preferred)
   - JSON file fallback for simple setups
   - Dynamic updates without restart required
   - Manages bot configs, access control, and user projects

3. **Session Manager** (`session_manager.py`)
   - Maintains conversation state across requests
   - Stores user sessions in database for continuity
   - Supports project-based routing configuration

4. **Forward Service** (`forward.py`)
   - Routes messages to Agent Studio endpoints
   - Handles HTTP communication with external services
   - Processes responses and manages errors

5. **Message Splitter** (`sender.py`)
   - Automatically splits long messages (>2000 chars) for WeChat Work
   - Handles text and image messages
   - Implements retry logic for failed sends

6. **Tunnel Server** (`tunnel.py`)
   - Integrates tunely for WebSocket tunneling
   - Enables external access to internal network services
   - Token-based authentication for secure connections

7. **Database Layer** (`database.py`, `models.py`)
   - SQLAlchemy 2.0 ORM with async support
   - Supports both SQLite (dev) and MySQL (prod)
   - Alembic for schema migrations

### Database Models

Located in `forward_service/models.py`:

- **Chatbot**: Bot configurations (name, target_url, access_mode, enabled)
- **ChatAccessRule**: Whitelist/blacklist rules per bot
- **UserProjectConfig**: User-specific project routing configurations
- **UserSession**: Session persistence for conversation continuity
- **ForwardLog**: Request/response logging for debugging
- **ProcessingSession**: Prevents concurrent processing of same message
- **ChatInfo**: Tracks chat type (group vs single chat)
- **SystemConfig**: Global system configuration

### Route Organization

- `/` - Root health check
- `/health` - System health check
- `/callback` - WeChat Work webhook endpoint (primary message entry)
- `/admin` - Admin API for configuration management
  - `/admin/bots` - Bot CRUD operations
  - `/admin/logs` - Request logs
  - `/admin/tunnels` - Tunnel management UI
- `/ws/tunnel` - WebSocket tunnel endpoint


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeffkit/as-dispatch](https://github.com/jeffkit/as-dispatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
