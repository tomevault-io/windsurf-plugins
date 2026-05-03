---
trigger: always_on
description: Express.js blog application built with TypeScript following **Clean Architecture** principles with clear separation of layers and SOLID design patterns.
---

# Project Guidelines

## Overview
Express.js blog application built with TypeScript following **Clean Architecture** principles with clear separation of layers and SOLID design patterns.

## Code Style
- **TypeScript**: Strict mode enabled (`tsconfig.json`)
- **Target**: ES6, CommonJS modules
- **Controllers**: Class-based controllers with explicit method binding (see [src/controllers/index.ts](../src/controllers/index.ts))
- **Types**: Custom interfaces in `src/types/index.ts` for shared type definitions
- **Error Handling**: Environment-aware error responses (development shows stack traces)
- **Naming**: Use descriptive names following single responsibility principle

## Architecture (Clean Architecture)
- **Layers Structure**:
  - **Domain Layer**: Entities and business logic (pure TypeScript, no framework dependencies)
  - **Use Cases Layer**: Application business rules and orchestration
  - **Interface Adapters**: Controllers, presenters, and gateways
  - **Frameworks & Drivers**: Express routes, database, external services
- **Dependency Rule**: Dependencies point inward (frameworks depend on use cases, not vice versa)
- **Separation of Concerns**: 
  - `app.ts`: Express app configuration and middleware setup
  - `server.ts`: Server initialization and port binding
  - Controllers only handle HTTP concerns (request/response)
  - Business logic lives in use cases/services
- **Route Organization**: 
  - **Separate route files by domain** (e.g., `routes/posts.routes.ts`, `routes/users.routes.ts`)
  - Each route file handles a single entity/feature
  - Centralized registration in `routes/index.ts`
- **Controller Pattern**: Use class-based controllers with `.bind(controllerInstance)` when registering routes
- **Middleware Order**: JSON/urlencoded parsing → routes → error handler (always last)

## Docker & Database
- **PostgreSQL in Docker**: Database runs in Docker container, application runs locally for development
  - Use `npm run docker:up` to start PostgreSQL container
  - Use `npm run docker:down` to stop container
  - Data persists in named volume `postgres_data`
- **Application Runtime**: App runs locally (not containerized) for fast hot-reload during development
- **Environment Variables**: Managed via `.env` file (never commit this file)
  - Use `.env.example` as template for required variables
  - Load with `dotenv/config` at the top of `server.ts`
- **Database Connection**: Initialized in `server.ts` startup, closed on graceful shutdown
- **Health Checks**: `/health` endpoint verifies database connectivity

## Prisma Conventions
- **Schema Location**: `prisma/schema.prisma` contains all models and configuration
- **Client Generation**: Always run `npm run prisma:generate` after schema changes
  - Prisma Client is generated in `node_modules/.prisma/client`
  - Type-safe query builder is available via `PrismaClient`
- **Database Access**: 
  - Use singleton `database` instance from `src/config/database.ts`
  - Export `prisma` for direct use in repositories
  - Follow interface pattern: `IDatabase` for dependency injection
- **Migrations Workflow**:
  - Development: `npm run prisma:migrate` (creates migration files)
  - Prototyping: `npm run db:push` (skip migrations, faster iteration)
  - Production: Apply migrations via CI/CD pipeline
- **Prisma Studio**: Use `npm run prisma:studio` for database GUI (localhost:5555)
- **Model Conventions**:
  - Use `@id`, `@default(autoincrement())` for primary keys
  - Add `createdAt DateTime @default(now())` and `updatedAt DateTime @updatedAt` to all models
  - Use descriptive relation names
  - Follow naming: models in PascalCase, fields in camelCase
- **Repository Pattern**: 
  - Create repository interfaces in domain layer
  - Implement repositories using `prisma` client from config
  - Never use `PrismaClient` directly in use cases or controllers
  - Example: `PostRepository` implements `IPostRepository` and injects `prisma`

## Git Worktree Support

> **⚠️ MANDATORY RULE - DO NOT SKIP**
> 
> Before ANY development work, you MUST run the appropriate setup based on your environment.
> Failure to do so will cause port conflicts and database errors.

- **Environment Detection**: Check if running in a worktree:
  ```bash
  git rev-parse --git-dir
  ```
  - If output contains `/worktrees/` → **You are in a worktree**
  - Otherwise → **You are in the main repository**

### REQUIRED Setup Steps

**🔴 IN A WORKTREE (MANDATORY):**
```bash
./scripts/setup-worktree.sh   # MUST run first - configures ports/schema
npm install
npm run prisma:migrate
npm run dev
```

**🟢 IN MAIN REPOSITORY:**
```bash
cp .env.example .env          # Copy defaults
npm install
npm run docker:up             # Start shared PostgreSQL
npm run prisma:migrate
npm run dev
```

### What setup-worktree.sh Does (Automatic)
- Copies `.env.example` → `.env`
- Finds available port (3000-3099) and sets `PORT`
- Generates unique PostgreSQL schema from branch name (`feature/auth` → `feature_auth`)
- Updates `DATABASE_URL` with isolated schema
- Creates `docker-compose.override.yml` if shared PostgreSQL not available

### Database Modes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [argentinaluiz/test-auto-copilot](https://github.com/argentinaluiz/test-auto-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
