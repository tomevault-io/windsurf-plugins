---
trigger: always_on
description: This document is a guide for AI agents working on the GraphJin codebase. It outlines the architectural patterns, coding conventions, and common tasks associated with maintaining and extending this library.
---

# GraphJin Agent Guide

This document is a guide for AI agents working on the GraphJin codebase. It outlines the architectural patterns, coding conventions, and common tasks associated with maintaining and extending this library.

## Architectural Overview

GraphJin is a compiler that turns GraphQL into database queries. For SQL databases, it generates SQL. For MongoDB, it generates a JSON DSL that is translated to aggregation pipelines. It is NOT a typical ORM or resolver-based GraphQL server.

-   **Core Philosophy**: Push as much work as possible to the database.
-   **No Resolvers**: Data fetching is done via a single generated SQL query. Do not add resolvers for database fields.
-   **Schema Driven**: The database schema (`sdata`) is the source of truth.

## Directory Structure & Responsibilities

| Path | Component | Responsibility |
| :--- | :--- | :--- |
| `core/api.go` | **Public API** | The only entry point for users. Changes here are breaking. |
| `core/core.go` | **Engine** | Internal orchestration, initialization, and state management. |
| `core/internal/sdata` | **Schema** | Metadata about tables, columns, and relationships. Graph traversal logic. |
| `core/internal/qcode` | **IR Compiler** | Front-end compiler. Parses GraphQL -> `QCode` (Intermediate Representation). |
| `core/internal/psql` | **SQL Compiler** | Back-end compiler. `QCode` -> SQL. Handles dialect differences. |
| `core/internal/dialect` | **Dialect Interface** | Database-specific SQL generation methods. Each dialect implements this interface. |
| `core/internal/graph` | **GraphQL Parser** | Lexer and parser for GraphQL input. Builds AST. |
| `core/internal/jsn` | **JSON Processing** | High-performance JSON parsing and filtering utilities. |
| `serv/` | **HTTP Service** | Standalone server with REST, GraphQL, and WebSocket APIs. |
| `auth/` | **Authentication** | Auth providers: JWT, Auth0, Firebase, Rails session. |
| `cmd/` | **CLI Tool** | Command-line interface for migrations, deployment, and management. |
| `conf/` | **Configuration** | YAML-based config loading and validation. |
| `wasm/` | **WebAssembly** | WASM build for NodeJS integration. |
| `mongodriver/` | **MongoDB Driver** | Custom database/sql-compatible driver for MongoDB. Translates JSON DSL to aggregation pipelines. |

## Build Commands

```bash
make build    # Build for current platform
make test     # Run tests (requires Docker for database containers)
make lint     # Lint code
make gen      # Generate code (stringer, etc.)
```

## Coding Guidelines

### 1. Adding New SQL Features
To add support for a new SQL feature (e.g., a new aggregation or function):
1.  **Update `qcode`**: meaningful changes often start here. Ensure the new feature can be represented in the `QCode` struct (`core/internal/qcode/qcode.go`).
2.  **Update `psql`**: Implement the SQL generation logic in `core/internal/psql/query.go` (or `mutate.go` for writes).
3.  **Tests**: Add a test case in `core/internal/psql/tests`. These tests compare the generated SQL against an expected string.
4.  **Dialect Compatibility**: If the feature syntax varies by database (Postgres vs MySQL):
    -   Add a method to the `Dialect` interface in `core/internal/dialect`.
    -   Implement it in `postgres.go` and `mysql.go`.
    -   Return an error from the implementation if the feature is not supported by that dialect.
    -   **DO NOT** use `if dialect == ...` checks in the shared `psql` logic.

### 2. MCP Maintenance

When adding new features to GraphJin (operators, syntax, capabilities), remember to update the MCP syntax documentation:

-   **File**: `serv/mcp_syntax.go`
-   **What to update**:
    -   `FilterOperators` struct - add new filter operators
    -   `querySyntaxReference` - add operator lists and syntax descriptions
    -   `queryExamples` - add example queries demonstrating new features
-   **Why**: AI assistants using MCP call `get_query_syntax` to learn available operators. Undocumented operators won't be used by AI agents.

### 3. Source Capability Maintenance

When adding a new `sources[].capabilities` key, add it first to the central registry in `core/sourcecap`. Do not introduce ad hoc capability strings in catalog, security, MCP, or source-default code.

GraphJin-owned system and workflow capabilities are not source capabilities. Add those to `core/featurecap` and enforce them through the centralized system-root policy.

-   **What the registry owns**: canonical source kinds, capability keys, mode defaults, action, severity, enforcement type, read-only behavior, summaries, recommendations, and examples.
-   **What subsystems own**: actual runtime enforcement hooks for their surface (for example CodeSQL, filesystem read-only, control-plane tables, or MCP tools).
-   **Tests**: registry, catalog, security, config validation, and permission tests must pass. If a capability is not runtime-enforced yet, mark it as `config_audit`.

### 4. Modifying Schema Discovery
If you need to change how GraphJin discovers tables or relationships:
-   Focus on `core/internal/sdata/schema.go` and `tables.go`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dosco/graphjin](https://github.com/dosco/graphjin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
