---
trigger: always_on
description: PuppyOne is a **cloud file system built for AI Agents**, centered around two core pillars: **Connect** and **Collaborate**.
---

# PuppyOne (ContextBase)

## Overview

PuppyOne is a **cloud file system built for AI Agents**, centered around two core pillars: **Connect** and **Collaborate**.

It aggregates information scattered across various sources into a unified Context Space, while providing a complete infrastructure for multi-party collaboration between humans and agents — authentication, access control, version history, audit logging, and backup/rollback. Through the file system, bash, and the MCP protocol, any agent can read and write this ContextBase just like a local file system.

### Connect

- **Multi-source data connectors** — OAuth connectors for 15+ platforms including Notion, GitHub, Gmail, Google Drive, Linear, Airtable, and more; also supports URL scraping, database connections, and custom scripts
- **Bidirectional local folder sync** — Real-time sync between local directories and the cloud Context Space via MUT protocol, powered by a background daemon
- **MCP protocol exposure** — Generates standard MCP interfaces for each agent or endpoint; any MCP-compatible client (Claude Desktop, Cursor, etc.) can connect directly
- **Code sandbox** — Securely execute code in isolated Docker/E2B containers; agents can invoke sandbox endpoints remotely

### Collaborate

- **Authentication & access control** — JWT for human users + Access Key for machine authentication; agent-level node access permissions
- **Version history & rollback** — File-level version management, arbitrary version diff comparison, one-click rollback; folder-level snapshots
- **Audit logging** — Records all operations (who did what to which node, and when), fully traceable
- **Collaborative editing** — Checkout/commit workflow, locking mechanism, conflict detection and resolution
- **Structured data management** — Cloud file system (folders/JSON/Markdown/files), JSON Pointer table operations

### Platform

- **Agent management** — Create agents, bind tools, control access scope, SSE streaming chat
- **Full CLI coverage** — Every operation available via command line, enabling AI coding tools like Claude Code to drive the platform directly
- **Unified access management** — All access types (sync/agent/MCP/sandbox/filesystem) consolidated into a single `access_points` table with a single entry point

## Active Development Directories

- **`backend/`** — Python (FastAPI) backend service
- **`frontend/`** — Next.js frontend application
- **`cli/`** — Node.js command-line tool (Commander.js)
- **`sandbox/`** — Docker sandbox environment (JSON editing / code execution)

## Deprecated Directories (do not modify)

- `PuppyEngine`, `PuppyFlow`, `PuppyStorage`, `tools`

---

## Backend

- **Language**: Python 3.12+
- **Framework**: FastAPI + Uvicorn (ASGI)
- **Package manager**: uv (`pyproject.toml`)
- **Database**: Supabase (PostgreSQL)
- **Storage**: AWS S3 / LocalStack
- **LLM gateway**: LiteLLM
- **Task queue**: ARQ (Redis)
- **Logging**: Loguru

### Directory Structure

```
backend/
├── src/
│   ├── main.py                # App entrypoint & lifespan
│   ├── config.py              # Global config (Pydantic Settings)
│   │
│   ├── mut_engine/            # Mut version engine (core write funnel)
│   │   ├── write_service.py   #   Single entry point for all content writes
│   │   ├── compat_service.py  #   Legacy CollaborationService compatibility layer
│   │   ├── repo_manager.py    #   Server-side repo lifecycle
│   │   ├── server_repo.py     #   PuppyOneServerRepo (S3 + Supabase adapter)
│   │   ├── ops.py             #   MutOps — unified tree operations entry point
│   │   ├── ephemeral_client.py #  MutEphemeralClient — in-process clone→push
│   │   ├── protocol_router.py #   MUT wire protocol (/clone /push /pull /negotiate)
│   │   ├── collab_router.py   #   Collab API (checkout/commit/versions/rollback/diff)
│   │   ├── audit_router.py    #   Audit log API
│   │   ├── audit_repository.py#   Audit log data access
│   │   ├── schemas.py         #   All Mut data types (Mutation, CommitResult, etc.)
│   │   ├── dependencies.py    #   FastAPI DI factories
│   │   ├── auth.py            #   MUT protocol auth (Access Key)
│   │   └── backends/          #   Storage backend adapters
│   │       ├── s3_storage.py  #     S3 object store
│   │       ├── supabase_history.py # mut_commits table
│   │       └── supabase_audit.py   # audit_logs table
│   │
│   ├── content/               # Content node tree (folder/JSON/MD/file)
│   │   └── table/             #     Structured data tables (JSON Pointer)
│   ├── tool/                  # Tool registration & search index
│   │
│   ├── connectors/            # Access types
│   │   ├── manager/           #   Unified access CRUD (connections table)
│   │   ├── datasource/        #   SaaS data source providers (Gmail/GitHub/Notion/...)
│   │   │   ├── gmail/         #     Gmail connector
│   │   │   ├── github/        #     GitHub connector
│   │   │   ├── google_drive/  #     Google Drive connector
│   │   │   ├── google_docs/   #     Google Docs connector
│   │   │   ├── google_sheets/ #     Google Sheets connector
│   │   │   ├── google_calendar/ #   Google Calendar connector
│   │   │   ├── google_search_console/ # GSC connector

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [puppyone-ai/puppyone](https://github.com/puppyone-ai/puppyone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
