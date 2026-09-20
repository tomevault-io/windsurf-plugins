---
trigger: always_on
description: > **AI assistants working on this codebase**: the canonical Version Engine
---

# PuppyOne (ContextBase)

> **AI assistants working on this codebase**: the canonical Version Engine
> architecture is in
> [`docs/architecture/01-version-engine.md`](docs/architecture/01-version-engine.md).
> PuppyOne is Git-native at the version layer: stock `git` talks to
> `https://<host>/git/ap/<access_key>.git`, while Web/API/`puppyone fs`
> writes converge through the Product Operation Adapter. Do not introduce the
> removed legacy wire protocol, external version package, or old source naming.

## Overview

PuppyOne is a **cloud file system built for AI Agents**, centered around two core pillars: **Connect** and **Collaborate**.

It aggregates information scattered across various sources into a unified Context Space, while providing a complete infrastructure for multi-party collaboration between humans and agents — authentication, access control, version history, audit logging, and backup/rollback. Through the file system, bash, and the MCP protocol, any agent can read and write this ContextBase just like a local file system.

### Connect

- **Multi-source data connectors** — OAuth connectors for 15+ platforms including Notion, GitHub, Gmail, Google Drive, Linear, Airtable, and more; also supports URL scraping, database connections, and custom scripts
- **Bidirectional local folder sync** — Real-time sync between local directories and the cloud Context Space via Git Remote and Puppyone CLI entry points
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
│   ├── version_engine/        # Git-native Version Engine (core write funnel)
│   │   ├── adapters/
│   │   │   ├── git/           #   Git smart-HTTP protocol boundary
│   │   │   └── operations/    #   ProductOperationAdapter for Web/API/CLI
│   │   ├── application/       #   transaction engine, merge policy, Git objects
│   │   ├── domain/            #   write/conflict intents
│   │   ├── routers/           #   content, history, conflict, AP-FS, websocket
│   │   ├── server/            #   repo manager, Supabase/S3 adapters, auth
│   │   └── services/          #   tree reader/splice, hooks, outbox, GC
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
│   │   │   ├── url/           #     URL/web page connector
│   │   │   └── _base.py       #     BaseConnector & ConnectorSpec
│   │   ├── filesystem/        #   Bidirectional local folder sync via Git Remote / CLI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [puppyone-ai/puppyone-cloud](https://github.com/puppyone-ai/puppyone-cloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
