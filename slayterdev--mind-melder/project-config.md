---
trigger: always_on
description: **Mind Melder** is a self-hosted productivity tool following a clean three-tier architecture:
---

# Copilot Instructions – Mind Melder

## Architecture Overview

**Mind Melder** is a self-hosted productivity tool following a clean three-tier architecture:

```
Frontend (React/Vite) → API (Express) → Data Layer (Drizzle ORM + PostgreSQL)
                            ↓
                      LLM Providers (OpenAI/Anthropic/Ollama)
```

**Monorepo structure** (pnpm workspaces):
- `apps/api` – Express REST API with versioned endpoints (`/api/v1/*`)
- `apps/web` – React frontend using Vite + Tailwind CSS
- `packages/database` – Drizzle ORM schema, migrations, and repository pattern
- `packages/llm` – Provider-agnostic LLM abstraction layer
- `packages/types` – Shared TypeScript types with Zod validation

**Core workflow**: Users quick-capture thoughts → stored as unorganized captures → LLM organizes into structured notes + actionable todos → Today Sheet prioritizes daily work

## Development Workflow

**Prerequisites**: Copy `.env.example` to `.env` and configure at least one LLM provider.

```bash
# Install dependencies
pnpm install

# Development (use Tilt for Docker-based dev matching production)
tilt up                    # Starts postgres, api, web containers + manual script resources
tilt down                  # Stop all services

# Or run services directly with pnpm (requires local postgres)
pnpm dev                   # Starts both api (port 3000) and web (port 5173) in parallel

# Database operations
docker compose up -d postgres  # Start PostgreSQL only
pnpm db:migrate               # Run pending migrations
pnpm db:generate              # Create new migration from schema changes
pnpm db:studio                # Open Drizzle Studio at http://localhost:4983

# Desktop app (Electron)
pnpm desktop:dev               # Start API + Electron app
pnpm desktop:build             # Build desktop app for current platform
pnpm desktop:build:mac         # Build for macOS
pnpm desktop:build:win         # Build for Windows
pnpm desktop:build:linux       # Build for Linux

# Code quality
pnpm lint                      # Run ESLint
pnpm format                    # Format with Prettier

# Testing scripts (manual Tilt resources or run directly)
./scripts/test-api.sh          # Test all API endpoints
./scripts/test-organization.sh # Test LLM organization flow
./scripts/test-today-sheet-api.sh # Test Today Sheet generation
./scripts/clear-db.sh          # Clear all data

# Production build
docker compose build           # Build all containers
docker compose up -d           # Start production stack
```

**Tilt resources**: `postgres`, `api`, `web` auto-start. Scripts (`script-clear-db`, `script-test-api`, etc.) are manual triggers.

## Repository Pattern (Critical)

All database access goes through repository classes in `packages/database/src/repositories/`. **Never write raw SQL queries in API routes or services.**

**Standard repository methods** (all entities):
```typescript
create(data)              // Insert new record
findById(id)              // Get by primary key
findByUserId(userId)      // Get all for user
update(id, data)          // Update record
delete(id)                // Delete record
```

**Entity-specific methods** (examples):
```typescript
CapturesRepository.findUnorganized(userId)  // Get captures where organized=null
CapturesRepository.markAsOrganized(id)      // Set organized timestamp
TodosRepository.findByStatus(userId, status)
TodosRepository.markAsCompleted(id)
TodaySheetsRepository.findMostRecentByDate(userId, date)
```

**See**: `packages/database/src/repositories/captures-repository.ts` for the canonical pattern.

## API Route Pattern

Routes follow a consistent factory pattern with dependency injection:

```typescript
// apps/api/src/routes/captures.ts
import { Router } from 'express';
import { CapturesRepository } from 'database';
import { createCaptureSchema } from 'types';
import { asyncHandler, validateBody, ApiError } from '../middleware/index.js';

export function createCapturesRouter(capturesRepo: CapturesRepository): Router {
  const router = Router();

  router.post('/', 
    validateBody(createCaptureSchema),
    asyncHandler(async (req, res) => {
      const userId = 'test-user-1'; // TODO: Auth in future milestone
      const capture = await capturesRepo.create({ ...req.body, userId });
      res.status(201).json(capture);
    })
  );

  return router;
}
```

**Key patterns**:
- Use `asyncHandler()` wrapper for all async routes (auto-catches errors)
- Validate requests with Zod schemas from `packages/types` via `validateBody()`/`validateQuery()`
- Throw `ApiError(statusCode, message)` for expected errors
- Repository instances injected as parameters to router factory functions
- Routes registered in `apps/api/src/index.ts` with `/api/v1` prefix

## LLM Provider Abstraction

LLM logic is **swappable** via factory pattern in `packages/llm/`:

```typescript
// Usage in services
import { ProviderFactory } from 'llm';

const provider = ProviderFactory.createFromEnv(); // Reads LLM_PROVIDER env var
const result = await provider.generateTodaySheet({ captures, existingTodos, template });
```

**Providers**: OpenAI (default: gpt-4o-mini), Anthropic (default: claude-3-5-sonnet-20241022), Ollama (local models)

**Configure in `.env`**:
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SlayterDev/Mind-Melder](https://github.com/SlayterDev/Mind-Melder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
