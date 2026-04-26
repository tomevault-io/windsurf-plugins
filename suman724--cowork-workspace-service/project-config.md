---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

`cowork-workspace-service` is the canonical backend store for agent-produced artifacts and session conversation history. It enables history browsing, cross-device access, and session continuation.

## Tech Stack

Python, FastAPI, PynamoDB/boto3 (DynamoDB + S3), Pydantic models from `cowork-platform`.

## API Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| `POST` | `/workspaces` | Create/resolve workspace (called by Session Service) |
| `POST` | `/workspaces/{id}/artifacts` | Upload artifact (tool output, file diff, or session_history) |
| `GET` | `/workspaces/{id}/artifacts/{artifactId}` | Retrieve artifact |
| `GET` | `/workspaces/{id}/sessions` | List sessions in workspace (paginated: `limit`, `nextToken`) |
| `GET` | `/workspaces/{id}/sessions/{sessionId}/history` | Get conversation messages for a session |
| `GET` | `/workspaces?tenantId=...&userId=...` | List workspaces for a user |
| `DELETE` | `/workspaces/{id}` | Delete workspace and all artifacts |
| `DELETE` | `/workspaces/{id}/sessions/{sessionId}` | Delete all artifacts for a session |
| `POST` | `/workspaces/{id}/files` | Upload file (multipart, `path` query param) — cloud only |
| `GET` | `/workspaces/{id}/files` | List files in workspace — cloud only |
| `GET` | `/workspaces/{id}/files/{path}` | Download file — cloud only |
| `DELETE` | `/workspaces/{id}/files/{path}` | Delete file — cloud only |

## Dual Storage Architecture

| Store | Technology | Holds |
|-------|-----------|-------|
| Metadata Store | DynamoDB | Workspace records, artifact metadata, session lists |
| Artifact Store | S3 | Raw artifact content (binary, text, JSON payloads) |

## Workspace Scopes

- `local`: Maps to a project directory on client. Reused across sessions. Resolved by `(tenantId, userId, localPath)`.
- `general`: Single-use per session. Always creates a new workspace.
- `cloud`: S3-backed workspace for sandbox sessions. Always creates new. Gets `s3_workspace_prefix = "{workspaceId}/workspace-files/"`. Supports file CRUD via `/files` endpoints.

## Session History

One `session_history` artifact per session. Each task completion **overwrites** the previous snapshot with a complete up-to-date conversation thread. This is the canonical store — the Desktop App always reads from here.

## Artifact Types

| Type | Produced by | Content |
|------|------------|---------|
| `session_history` | Agent Host (after each task) | Full ConversationMessage[] thread |
| `tool_output` | Tool Runtime (shell output >10KB) | Raw tool output text |
| `file_diff` | Tool Runtime (after file writes) | Unified diff |

## DynamoDB Tables

**`{env}-workspaces`** — PK: `workspaceId`

| GSI | PK | SK | Use |
|-----|----|----|-----|
| `tenantId-userId-index` | `tenantId` | `userId` | List workspaces for user |
| `localpath-lookup-index` | `localPathKey`* | — | Idempotent resolve of local workspaces |

*`localPathKey` = `{tenantId}#{userId}#{localPath}` — populated only for `local`-scoped workspaces.

**`{env}-artifacts`** — PK: `workspaceId`, SK: `artifactId`

| GSI | PK | SK | Use |
|-----|----|----|-----|
| `sessionId-type-index` | `sessionId` | `artifactType#createdAt` | Fetch session_history, list tool outputs |

## S3 Bucket: `{env}-workspace-artifacts`

Object key: `{workspaceId}/{sessionId}/{artifactType}/{artifactId}`

## Repository Pattern

```
WorkspaceService
  └── WorkspaceRepository (interface)
        ├── DynamoWorkspaceRepository   ← production
        └── InMemoryWorkspaceRepository ← unit tests
```

Integration tests use LocalStack (DynamoDB + S3 together): `AWS_ENDPOINT_URL=http://localhost:4566`

## Design Doc

Full specification: `cowork-infra/docs/services/workspace-service.md`

---

## Engineering Standards

### Project Structure

```
cowork-workspace-service/
  CLAUDE.md
  README.md
  Makefile
  Dockerfile
  pyproject.toml
  .python-version             # 3.12
  .env.example
  src/
    workspace_service/
      __init__.py
      main.py                 # FastAPI app factory with lifespan
      config.py               # pydantic-settings: Settings class
      dependencies.py         # FastAPI Depends providers (repos, S3 client)
      routes/
        __init__.py
        health.py             # GET /health, GET /ready
        workspaces.py         # Workspace CRUD endpoints
        artifacts.py          # Artifact upload/download endpoints
        files.py              # Workspace file CRUD endpoints (cloud only)
      services/
        __init__.py
        workspace_service.py  # Workspace business logic
        artifact_service.py   # Artifact business logic (DynamoDB metadata + S3 content)
        file_service.py       # Workspace file operations (cloud scope)
      repositories/
        __init__.py
        base.py               # WorkspaceRepository, ArtifactRepository Protocols
        dynamo_workspace.py   # DynamoDB workspace repository
        dynamo_artifact.py    # DynamoDB artifact metadata repository
        s3_artifact_store.py  # S3 artifact content storage
        memory.py             # InMemory implementations for unit tests
      models/
        __init__.py

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/suman724) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
