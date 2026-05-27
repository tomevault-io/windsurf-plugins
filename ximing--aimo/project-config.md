---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AIMO is a full-stack AI-powered note-taking and knowledge management system. It's a pnpm monorepo with a React 19 frontend and Express.js backend using a dual-database architecture: MySQL (via Drizzle ORM) for relational data and LanceDB for vector search.

## Development Commands

### Setup

```bash
# Install dependencies (requires pnpm 10.22.0+)
pnpm install

# Copy environment variables
cp .env.example .env
# Edit .env - required: JWT_SECRET (32+ chars), OPENAI_API_KEY, MySQL credentials
```

### Development

```bash
# Start all apps (frontend + backend)
pnpm dev

# Start individual apps
pnpm dev:web      # Frontend only - Vite dev server at http://localhost:5173
pnpm dev:server   # Backend only - Express at http://localhost:3000

# Start Docker dependencies (MySQL, etc.)
pnpm dev:env
```

### Build

```bash
# Build all packages (DTO builds first due to Turbo dependency graph)
pnpm build

# Build specific apps
pnpm build:web
pnpm build:server
```

### Code Quality

```bash
pnpm lint           # ESLint across all packages
pnpm lint:fix       # Auto-fix lint issues
pnpm format         # Prettier format entire codebase
```

### Testing

```bash
# Run tests (server only - configured with Jest)
cd apps/server && pnpm test

# Run single test file
cd apps/server && pnpm test -- path/to/test.ts

# Run tests matching pattern
cd apps/server && pnpm test -- --testNamePattern="pattern"
```

### Database (Drizzle ORM)

```bash
# Must build first before generating migrations
cd apps/server && pnpm build
cd apps/server && pnpm migrate:generate   # Generate migration from schema changes
cd apps/server && pnpm migrate            # Run migrations manually
cd apps/server && pnpm migrate:studio     # Open Drizzle Studio
cd apps/server && pnpm migrate:data       # Migrate data from LanceDB to MySQL (one-time script)
```

### Cleanup

```bash
pnpm clean          # Clean dist directories
pnpm rm             # Remove all node_modules recursively
```

## Monorepo Architecture

### Package Structure

```
aimo/
├── apps/
│   ├── web/              # React 19 + Vite frontend
│   └── server/           # Express.js + TypeScript backend
├── packages/
│   └── dto/              # Shared Data Transfer Objects (Rollup-built)
└── config/
    ├── config-typescript/# Shared tsconfig
    ├── eslint-config/    # Shared ESLint rules
    ├── jest-presets/     # Shared Jest configs
    └── rollup-config/    # Shared Rollup configs
```

Turbo orchestrates build order: `@aimo/dto` first, then `@aimo/web` and `@aimo/server` in parallel. Always import from `@aimo/dto` for shared types.

## Backend Architecture (apps/server)

### Key Technologies

- **Express.js** with routing-controllers (decorator-based routing)
- **TypeDI** for dependency injection
- **MySQL** via Drizzle ORM for relational data (users, memos, AI conversations, etc.)
- **LanceDB** for vector database (semantic search, embeddings)
- **OpenAI** for text embeddings (text-embedding-3-small default)
- **JWT** authentication with bcrypt
- **Multer** for file uploads

### Dual Database Architecture

The backend uses two databases with distinct responsibilities:

- **MySQL (Drizzle ORM)** — relational data: users, memos, attachments, categories, tags, AI conversations/messages, push rules, daily recommendations, memo relations
  - Schema in `src/db/schema/` — one file per table, all exported from `schema/index.ts`
  - Access via `getDatabase()` singleton from `src/db/connection.ts`
  - Migrations in `drizzle/` folder, auto-run on startup
  - **Always build before generating migrations** (drizzle-kit reads from `dist/`)
  - VARCHAR(191) limit for keys (MySQL utf8mb4 index limit)

- **LanceDB** — vector data: memo embeddings for semantic search
  - Abstraction in `src/sources/lancedb.ts`
  - Embeddings auto-generated via `EmbeddingService` on memo create/update
  - Supports local or S3 storage backends

### Server Startup Sequence

1. IOC container (TypeDI)
2. MySQL connection pool
3. Drizzle migrations
4. LanceDB initialization
5. Scheduler service
6. Express server

### Service Layer

Business logic in `src/services/`:

- `memo.service.ts` — CRUD + embedding generation
- `search.service.ts` — Vector similarity search via LanceDB
- `attachment.service.ts` — File storage (local/S3/OSS)
- `user.service.ts` — User management
- `ai.service.ts` / `ai-conversation.service.ts` — AI chat with conversation persistence
- `embedding.service.ts` — Text embedding generation (OpenAI)
- `multimodal-embedding.service.ts` — Image/video embeddings (DashScope/Qwen)
- `memo-relation.service.ts` — Directed memo relations and backlinks
- `category.service.ts`, `tag.service.ts` — Organization
- `recommendation.service.ts` — Daily content recommendations
- `scheduler.service.ts` — Cron jobs (DB optimization, push notifications)
- `push-rule.service.ts` — Push notification rules
- `ocr/` — OCR providers (Zhipu, etc.)
- `channels/` — Push notification channels (Feishu, Meow)
- `asr.service.ts` — Automatic speech recognition
- `explore.service.ts` — AI-powered content exploration

### Controllers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ximing/aimo](https://github.com/ximing/aimo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
