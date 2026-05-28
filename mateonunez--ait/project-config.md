---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Reference

### Common Commands

```bash
# Setup
corepack enable
pnpm install
pnpm start:services              # Start Docker services (PostgreSQL, Qdrant, Ollama, Redis, Langfuse)
pnpm migrate                     # Run database migrations
pnpm run --filter @ait/postgres db:seed  # Seed database with initial provider config

# Development
pnpm dev                         # Start all dev servers
pnpm build                       # Build all packages
pnpm test                        # Run tests (starts test Docker services automatically)
pnpm test:watch                  # Watch mode for tests

# Code Quality
pnpm lint                        # Check with Biome
pnpm lint:fix                    # Auto-fix issues
pnpm typecheck                   # Run TypeScript type checking

# Database
pnpm migrate                     # Run migrations (from root or postgres package)
pnpm run --filter @ait/postgres db:studio  # Open database UI
cd packages/infrastructure/postgres && pnpm db:generate  # Regenerate DB types

# Code Generation
pnpm generate:openapi            # Generate OpenAPI TypeScript types (connectors)

# Service Management
pnpm start:services              # Start all Docker services
pnpm stop:services               # Stop services
pnpm clean:services              # Stop and remove volumes
pnpm clean:all                   # Full cleanup (node_modules, dist, services)

# Testing Specific Packages
pnpm --filter @ait/core test
pnpm --filter @ait/connectors test
pnpm --filter @ait/ai-sdk test
```

### Running a Single Test

```bash
# Node.js native test runner - use --test-name-pattern
node --test --test-name-pattern="test name pattern" path/to/test-file.test.ts
```

## Architecture Overview

AIt is a monorepo with a layered architecture that follows this data flow:

```
OAuth → Connector → PostgreSQL → Scheduler/ETL → Embeddings → Qdrant → RAG → AI Response
```

### Key Layers

1. **Gateway** (`packages/gateway`)
   - Unified API gateway for all connectors
   - Handles OAuth 2.0 flows and token management
   - Provides REST endpoints: `/api/{provider}/auth`, `/api/{provider}/data`
   - Includes analytics, cache, and insights services
   - Uses Express with session-based authentication
   - OAuth credentials stored encrypted in database (not env vars)

2. **Connectors** (`packages/connectors`)
   - Modular OAuth 2.0 integrations: GitHub, Linear, Spotify, X, Notion, Slack, Google (Calendar, YouTube, Photos, Contacts)
   - Database-driven configuration via `connectorServiceFactory`
   - Shared base classes in `shared/` directory
   - Store implementations in `infrastructure/vendors/`
   - Domain logic in `domain/`
   - Each connector normalizes data to canonical `EntityType` values (e.g., `spotify_track`, `github_repository`)

3. **AI SDK** (`packages/infrastructure/ai-sdk`)
   - Custom Ollama integration (zero external AI SDK dependencies)
   - Direct HTTP API calls for full control
   - RAG with Qdrant vector search (multi-query retrieval)
   - Tool calling for connector searches
   - Streaming text generation
   - Embeddings with chunking and LRU caching
   - Provider registration for cache/analytics (used by Gateway)

4. **Scheduler** (`packages/infrastructure/scheduler`)
   - BullMQ + Redis-based job queue
   - Priority-based ETL scheduling (1=high, 3=low)
   - Configurable cron intervals (dev vs production)
   - Concurrent worker processing (default: 2)
   - Uses TypeScript decorators (requires reflect-metadata, .swcrc config)

5. **RetoVe (ETL)** (`packages/transformers/retove`)
   - Relational to Vector ETL pipeline
   - Extracts from PostgreSQL → generates embeddings → loads to Qdrant
   - Multiple embedding methods: Ollama SDK, LangChain, Python scripts
   - Automated via Scheduler or manual via `pnpm etl`

6. **Core** (`packages/core`)
   - Shared utilities, types, error handling
   - Result type for type-safe error handling (`Result<T, E>`, `ok()`, `err()`)
   - HTTP client (`requestJson`, `requestStream`)
   - Logger with correlation IDs
   - Zod-based validation
   - OpenAPI-generated integration types
   - Custom error classes: `AItError`, `RateLimitError`

7. **Infrastructure**
   - **PostgreSQL** (`packages/infrastructure/postgres`): Drizzle ORM, migrations in `src/migrations/`
   - **Qdrant** (`packages/infrastructure/qdrant`): Vector database client
   - **Storage** (`packages/infrastructure/storage`): S3/MinIO for binary assets (Google Photos)
   - **Store** (`packages/infrastructure/store`): Application data (conversations, feedback, goals)
   - **Redis** (`packages/infrastructure/redis`): Job queue and caching

8. **UI** (`packages/apps/`)
   - **UIt**: Main React web interface (Vite + React)
   - **Chat**: Landing page (React + Vite + Tailwind)

## Key Architectural Patterns

### Entity Types
All data is normalized with stable `__type` strings defined in `@ait/core` `EntityType`. This drives:
- Gateway API filtering/routing
- Qdrant collection routing in ETL
- UI entity pages


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mateonunez/ait](https://github.com/mateonunez/ait) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
