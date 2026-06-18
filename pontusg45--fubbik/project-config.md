---
trigger: always_on
description: This file provides context about the project for AI assistants.
---

# fubbik

This file provides context about the project for AI assistants.

> **New to this codebase?** Use the `fubbik-dev` skill — it has code patterns for every layer and pitfalls that will save you time.

## Git Rules

- **Always ask before pushing.** Never run `git push` without explicit user confirmation.

## Project Overview

- **Ecosystem**: Typescript
- **Purpose**: Local-first knowledge framework for storing, navigating, and evolving structured knowledge about codebases. Designed for both humans (web UI, graph visualization) and machines (CLI, API, VS Code extension, MCP server).

## Tech Stack

- **Runtime**: bun
- **Package Manager**: pnpm

### Frontend

- Framework: tanstack-start (SSR via entry-server.ts)
- CSS: tailwind
- UI Library: shadcn-ui (built on @base-ui/react — uses `render` prop, NOT `asChild`)
- Graph: @xyflow/react (React Flow)

### Backend

- Framework: elysia
- API Client: elysia eden treaty
- Validation: elysia `t` schema (NOT arktype — arktype was removed)
- Error Handling: effect (typed errors via Effect.tryPromise, tagged errors)

### Database

- Database: postgres (v18)
- ORM: drizzle
- Extensions: pgvector 0.8.2 (embeddings), pg_trgm (fuzzy text search)

### Authentication

- Provider: better-auth

### Additional Features

- Testing: vitest
- AI: Ollama (local LLM for enrichment + embeddings), OpenAI via Vercel AI SDK (optional, for summarization/suggestions)
- Embeddings: Ollama (nomic-embed-text) for local vector embeddings
- Logging: winston
- Observability: opentelemetry
- MCP: Model Context Protocol server for AI agent integration

## Project Structure

```
fubbik/
├── apps/
│   ├── web/         # Frontend application (TanStack Start)
│   ├── server/      # Backend API server (Elysia)
│   ├── cli/         # CLI application (Commander.js)
│   └── vscode/      # VS Code / Cursor extension
├── packages/
│   ├── api/         # API layer (Elysia routes, Eden types)
│   ├── auth/        # Authentication (Better Auth)
│   ├── config/      # Shared TypeScript config
│   ├── db/          # Database schema (Drizzle ORM)
│   ├── env/         # Environment validation
│   └── mcp/         # MCP server (AI agent tools)
└── docs/
    └── superpowers/
        ├── specs/   # Design specifications
        └── plans/   # Implementation plans
```

## Core Concepts

### Chunks

The central entity — discrete units of knowledge (conventions, architecture decisions, runbooks, etc.). Each chunk has:
- `title`, `content`, `type` (note, document, reference, schema, checklist)
- `tags` (via normalized join table with tag types)
- `scope` (JSONB key-value metadata)
- `appliesTo` (glob patterns linking chunks to file areas, via `chunk_applies_to` table)
- `fileReferences` (explicit bidirectional links to files/symbols, via `chunk_file_ref` table)
- `rationale`, `alternatives`, `consequences` (optional "why" decision context fields)
- `embedding` (768-dim vector for semantic search)
- `embeddingUpdatedAt` (tracks when embedding was last refreshed)
- AI-generated: `summary`, `aliases`, `notAbout`
- Health score (computed on-demand): freshness + completeness + richness + connectivity (0-100)
- Version history (append-only `chunk_version` table)

### Codebases & Workspaces

Chunks can be organized per-codebase. A codebase is identified by git remote URL (normalized) or local paths. Chunks can belong to multiple codebases or none (global).
- `codebase` table with `remoteUrl`, `localPaths`, `name`
- `chunk_codebase` join table (many-to-many)
- `workspace` table groups related codebases (e.g., frontend + backend + infra)
- `workspace_codebase` join table
- CLI auto-detects codebase via git remote
- Web UI has a codebase/workspace switcher in the nav

### Connections

Directed edges between chunks: `sourceId → targetId` with a `relation` type (related_to, part_of, depends_on, extends, references, supports, contradicts, alternative_to). Connections are global (not codebase-scoped), enabling cross-project knowledge linking.

### Templates

Built-in + user-created chunk templates (Convention, Architecture Decision, Runbook, API Endpoint). Built-in templates are seeded via SQL migration and are read-only. Users can duplicate and customize.

### Plans

The central unit of work. Each plan holds a description, linked requirements, structured analyze fields, and enriched tasks.

- `plan` table: `title`, `description` (markdown), `status` (`draft | analyzing | ready | in_progress | completed | archived` — labels only, ungated), `userId`, `codebaseId`, `completedAt`
- `plan_requirement` — many-to-many link to existing `requirement` entities at the plan level
- `plan_analyze_item` — discriminated table holding five kinds: `chunk`, `file`, `risk`, `assumption`, `question`, each with kind-specific metadata (severity for risks, verified flag for assumptions, answer for questions, line range for files)
- `plan_task` — enriched tasks with `title`, `description`, `acceptanceCriteria` (JSONB string array), `status`
- `plan_task_chunk` — many-to-many linking tasks to multiple chunks with a relation (`context | created | modified`)
- `plan_task_dependency` — task dependencies; marking a task `done` auto-unblocks dependents in `blocked` state

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pontusg45/fubbik](https://github.com/Pontusg45/fubbik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
