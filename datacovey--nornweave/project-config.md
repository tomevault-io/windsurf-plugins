---
trigger: always_on
description: NornWeave is an open-source, self-hosted Inbox-as-a-Service API for AI Agents. It provides a stateful email layer (Inboxes, Threads, History) and an intelligent layer (Markdown parsing, Semantic Search) for LLMs via REST or MCP.
---


# NornWeave - Cursor Rules

NornWeave is an open-source, self-hosted Inbox-as-a-Service API for AI Agents. It provides a stateful email layer (Inboxes, Threads, History) and an intelligent layer (Markdown parsing, Semantic Search) for LLMs via REST or MCP.

## Repository Structure

```
nornweave/
├── src/nornweave/              # Main Python package
│   ├── adapters/               # Email provider adapters (Mailgun, SES, SendGrid, Resend, SMTP/IMAP, demo)
│   ├── core/                   # Core utilities (config, exceptions, interfaces)
│   ├── models/                 # Pydantic/SQLAlchemy models (Event, Inbox, Message, Thread)
│   ├── huginn/                 # MCP Resources (read operations for AI agents)
│   ├── muninn/                 # MCP Tools (write operations for AI agents)
│   ├── skuld/                  # Outbound layer (rate limiting, scheduling, sending, webhooks)
│   ├── urdr/                   # Storage layer (database adapters, migrations)
│   │   ├── adapters/           # PostgreSQL and SQLite adapters
│   │   └── migrations/         # Alembic migration scripts
│   ├── verdandi/               # Ingestion (webhook + IMAP, parsing, threading, attachments, LLM summarization)
│   └── yggdrasil/              # FastAPI gateway
│       ├── middleware/         # Auth and logging middleware
│       └── routes/             # API routes (v1/, webhooks/, v1/demo when in demo mode)
├── clients/                    # SDK clients for NornWeave API
│   └── python/                 # Python client library (nornweave-client)
├── packages/                   # External platform integrations
│   └── n8n-nodes-nornweave/    # n8n community node for NornWeave
├── tests/                      # Test suite (fixtures/, integration/, unit/, e2e/)
├── web/                        # Hugo documentation website
│   └── content/docs/           # Documentation content
├── scripts/                    # Utility scripts (DB init, migrations, dev setup)
├── skills/                     # Distributable AI assistant skills for NornWeave
├── openspec/                   # OpenSpec artifacts (specs, changes, context for implementations)
│   ├── specs/                  # Feature specifications and design documents
│   └── changes/archive/        # Completed and archived changes
└── .github/                    # CI/CD workflows
```

## Architecture Overview

NornWeave uses a thematic architecture inspired by Norse mythology:

| Module | Name | Purpose |
|--------|------|---------|
| `urdr` | The Well | Storage layer - PostgreSQL/SQLite adapters, Alembic migrations |
| `verdandi` | The Loom | Ingestion - webhook + IMAP, HTML→Markdown, threading, LLM thread summarization |
| `skuld` | The Prophecy | Outbound - rate limiting, scheduling, email sending |
| `yggdrasil` | World Tree | FastAPI gateway - routes, middleware, API endpoints |
| `huginn` | Thought Raven | MCP Resources - read operations for AI agents |
| `muninn` | Memory Raven | MCP Tools - write operations for AI agents |

## Technology Stack

- **Python**: 3.14+ with full type hints
- **Web Framework**: FastAPI with async/await
- **Validation**: Pydantic v2 with `pydantic-settings`
- **Database**: SQLAlchemy 2.0 (async) with PostgreSQL (asyncpg) or SQLite (aiosqlite)
- **Migrations**: Alembic
- **AI Integration**: MCP SDK for Claude/Cursor integration
- **HTTP Client**: httpx (async)
- **Package Manager**: uv
- **Linting/Formatting**: Ruff
- **Type Checking**: MyPy (strict mode)
- **Testing**: pytest with pytest-asyncio
- **Documentation**: Hugo static site

## Development Guidelines

### General Principles
- **Type safety**: All functions must have complete type hints
- **Async-first**: Use async/await for all I/O operations
- **Documentation**: Docstrings for all public functions and classes
- **Testing**: Unit tests for business logic, integration tests for API endpoints
- **Security**: Never hardcode credentials; use environment variables

### Python Code Style
- Follow PEP 8 (enforced by Ruff)
- Line length: 100 characters
- Use double quotes for strings
- Use `pathlib.Path` over `os.path`
- Use comprehensions where they improve readability
- Prefer `httpx` for HTTP requests (async support)

### FastAPI Best Practices
- Use dependency injection via `Depends()`
- Define request/response models with Pydantic
- Use `HTTPException` for error responses with appropriate status codes
- Group routes in routers, version API endpoints (e.g., `/v1/`)
- Add OpenAPI documentation to endpoints

### Database Guidelines
- Use SQLAlchemy async sessions
- Define models in `src/nornweave/models/`
- Create Alembic migrations for all schema changes in `src/nornweave/urdr/migrations/`
- Use repository pattern for data access in `urdr/adapters/`
- Support both PostgreSQL (production) and SQLite (development/testing)

### MCP Integration
- Resources (read-only data) go in `huginn/`
- Tools (actions) go in `muninn/`
- Follow MCP SDK patterns for tool definitions
- Include clear descriptions for AI agent consumption

### Testing
- Place unit tests in `tests/unit/` mirroring source structure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataCovey/nornweave](https://github.com/DataCovey/nornweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
