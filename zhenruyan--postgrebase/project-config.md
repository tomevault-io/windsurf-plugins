---
trigger: always_on
description: This file provides context for AI coding agents (Cursor, Claude, Windsurf, etc.) working on the PostgreBase codebase. Read this before exploring or modifying code.
---

# PostgreBase — Agent Development Guide

This file provides context for AI coding agents (Cursor, Claude, Windsurf, etc.) working on the PostgreBase codebase. Read this before exploring or modifying code.

## Overview

PostgreBase 是 AI 原生的无代码 API 开发平台，基于 [PocketBase](https://pocketbase.io) 重构。内置 MCP (Model Context Protocol) 服务器，让 AI 工具直接操作数据。支持 **PostgreSQL**、**MySQL**、**SQLite** 三种数据库，混合缓存（**Redis** / 内存），100% 兼容 PocketBase API、Admin UI 和业务逻辑。

## Tech Stack

- **Language:** Go 1.26.2+ (builds with `CGO_ENABLED=0`)
- **HTTP Framework:** `github.com/labstack/echo/v5`
- **CLI:** `github.com/spf13/cobra`
- **Database Drivers:**
  - `github.com/lib/pq` — PostgreSQL
  - `github.com/go-sql-driver/mysql` — MySQL
  - `modernc.org/sqlite` — SQLite (pure Go, no CGO)
- **Caching:** `github.com/redis/go-redis/v9` (Redis) / built-in memory store
- **Frontend:** Svelte 3, svelte-spa-router, Vite 4, PocketBase JS SDK
- **Vendor:** `/vendor/` directory **must be preserved** — all dependencies are vendored for offline builds.

## Project Structure

```
postgrebase/
├── build/              # Server entry point (main.go)
├── postgrebase.go      # Root package: CLI setup, Config struct, Bootstrap
├── core/               # Application logic
│   ├── base.go         # BaseApp: cache init, app lifecycle
│   ├── db_postgresql.go # connectDB(): DSN parsing and driver detection
│   └── app.go          # App interface definition
├── daos/               # Data access objects (CRUD for all models)
│   ├── base.go         # Dao struct, ModelQuery, RunInTransaction
│   ├── record.go       # Record CRUD
│   ├── record_table_sync.go # Table schema sync (driver-aware DDL)
│   ├── table.go        # HasTable, TableColumns, TableInfo, TableIndexes (driver-aware)
│   ├── collection.go   # Collection queries
│   ├── admin.go        # Admin auth and queries
│   └── view.go         # SQL view management
├── models/             # Data models
│   ├── base.go         # BaseModel (Id, Created, Updated, RefreshId)
│   ├── record.go       # Record model
│   ├── collection.go   # Collection model
│   ├── admin.go        # Admin model (bcrypt password, JWT token key)
│   └── schema/
│       └── schema_field.go # SchemaField.ColDefinition(driverName) — driver-aware DDL
├── apis/               # HTTP API handlers
│   ├── base.go         # InitApi(): route registration, MCP route binding
│   ├── serve.go        # HTTP server, migration runner, startup banner
│   ├── record_crud.go  # Record CRUD endpoints
│   ├── admin.go        # Admin auth endpoints
│   ├── mcp_token.go    # MCP token management API (CRUD)
│   └── middlewares.go   # Auth middleware (RequireAdminAuth, etc.)
├── mcp/                # MCP (Model Context Protocol) server
│   ├── server.go       # JSON-RPC 2.0 core, method routing, tool schemas
│   ├── tools.go        # 8 MCP tools (CRUD + search for records/collections)
│   ├── resources.go    # 2 MCP resources (collections, settings)
│   ├── auth.go         # Token validation (JWT admin tokens + mcp_ prefixed tokens)
│   ├── transport_sse.go    # SSE + Streamable HTTP transport
│   └── transport_stdio.go  # Stdin/stdout transport
├── migrations/         # Database migrations (driver-aware SQL)
│   ├── 1640988000_init.go              # Core tables (admins, collections, params, externalAuths)
│   ├── 1691747914_add_cache_columns.go # Cache columns migration
│   ├── 1704067200_mcp_tokens.go        # MCP tokens collection
│   └── 1730000000_agent_runtime.go     # Agent sessions/messages/audit/project-config tables
├── agents/             # Embedded agent platform (orchestration over controlled tools)
│   ├── service.go      # Service facade (runtime, sessions, tools, run)
│   ├── runtime.go      # RunSession: vibecoding agent loop, naming, audit
│   ├── toolkit.go      # ToolRegistry + schema.*/data.* executors + ChartHint
│   ├── sdk_tools.go    # agent.ExternalTool adapter (project scope + write authz)
│   ├── authz.go        # RunOptions, write-approval policy, audit sink
│   ├── project_config.go # per-project overrides (§9.1)
│   ├── files.go        # record file ref → image content block (§6.2)
│   └── store_db.go     # DB-backed session/message store
├── cmd/                # CLI commands
│   ├── serve.go        # `serve` command
│   ├── admin.go        # `admin create` command
│   └── mcp.go          # `mcp` command (stdio transport)
├── dbx/                # Database query builder (fork of ozzo-dbx)
│   ├── builder.go      # BaseBuilder: CreateTable, AddColumn, DropColumn, etc.
│   ├── builder_sqlite.go # SqliteBuilder: SQLite-specific overrides
│   ├── builder_pgsql.go  # PgsqlBuilder: PostgreSQL-specific overrides
│   └── builder_mysql.go  # MysqlBuilder: MySQL-specific overrides
├── tools/              # Shared utilities
│   ├── security/       # JWT, bcrypt, random string generation
│   ├── types/          # DateTime (custom time type with Scan/MarshalJSON)
│   ├── search/         # Filter/sort/search provider
│   ├── migrate/        # Migration runner
│   └── list/           # Slice/string helpers
├── ui/                 # Admin UI (Svelte SPA)
│   ├── src/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhenruyan/postgrebase](https://github.com/zhenruyan/postgrebase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
