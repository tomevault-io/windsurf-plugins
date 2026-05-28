---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HyperFix is a self-hosted project management platform built with simplicity and performance as core principles. The codebase is organized as a **pnpm monorepo** with TurboRepo.

**Key Philosophy**: Features exist to solve real problems, not to impress. Avoid over-engineering - keep solutions simple and focused. Don't add features, refactoring, or improvements beyond what was asked.

## Development Commands

### Getting Started
```bash
# Install dependencies (uses pnpm)
pnpm install

# Start all development servers (API + web)
pnpm dev

# Lint and auto-fix code (Biome)
pnpm lint

# Build all packages
pnpm build
```

### API-Specific Commands
```bash
# Run API in development mode
pnpm --filter @hyperfix/api dev

# Build API
pnpm --filter @hyperfix/api build

# Generate database migrations (after schema changes)
pnpm --filter @hyperfix/api db:generate

# Run database migrations (auto-runs on API startup)
pnpm --filter @hyperfix/api db:migrate

# Open Drizzle Studio (database GUI)
pnpm --filter @hyperfix/api db:studio

# Lint API code
pnpm --filter @hyperfix/api lint
```

### Web-Specific Commands
```bash
# Run web app in development mode
pnpm --filter @hyperfix/web dev

# Build web app for production
pnpm --filter @hyperfix/web build

# Preview production build
pnpm --filter @hyperfix/web preview

# Lint web code
pnpm --filter @hyperfix/web lint
```

## Architecture Overview

### Monorepo Structure
```
hyperfix/
├── apps/
│   ├── api/          # Backend API (Hono/Node.js/PostgreSQL)
│   ├── web/          # Frontend app (React/Vite/TanStack)
│   └── docs/         # Documentation site (Next.js)
├── packages/
│   ├── email/        # Email utilities
│   ├── libs/         # Shared libraries
│   └── typescript-config/  # TypeScript configurations
└── charts/           # Kubernetes Helm charts
```

### Technology Stack

**Backend (API)**
- Framework: Hono (lightweight web framework)
- Database: PostgreSQL with Drizzle ORM
- Authentication: Better Auth
- Validation: Valibot (Zod is also present, used by Better Auth and some schemas)
- API Documentation: OpenAPI (hono-openapi)
- IDs: CUID2 (via @paralleldrive/cuid2)

**Frontend (Web)**
- Framework: React 19+
- Routing: TanStack Router (file-based)
- Data Fetching: TanStack Query (React Query)
- Build Tool: Vite
- Styling: Tailwind CSS v4
- State Management: Zustand
- UI Components: Radix UI primitives

### Key Architectural Patterns

**Backend API Structure**
- Routes organized by feature in `apps/api/src/{feature}/`
- Controller pattern: business logic extracted to `{feature}/controllers/`
- All routes use OpenAPI decorators (`describeRoute`)
- All inputs validated with Valibot schemas
- Migrations auto-run on API startup

**Frontend Structure**
- File-based routing in `apps/web/src/routes/`
- Query hooks in `apps/web/src/hooks/queries/`
- Mutation hooks in `apps/web/src/hooks/mutations/`
- API fetchers in `apps/web/src/fetchers/{feature}/`
- Components in `apps/web/src/components/`

**Database Schema Conventions**
- All tables use CUID2 for primary keys (`createId()`)
- Every table has `createdAt` and `updatedAt` timestamps
- Foreign keys always specify cascade behavior (`onDelete`, `onUpdate`)
- Indexes on frequently queried columns (especially foreign keys)
- Schema defined in `apps/api/src/database/schema.ts`
- Relations defined in `apps/api/src/database/relations.ts`

**Authentication Flow**
- Better Auth handles authentication
- User context available in Hono via `c.get("userId")`, `c.get("user")`, `c.get("session")`
- API keys supported via Bearer token
- Frontend uses Better Auth client from `@/lib/auth-client`

**Event System**
- Events published for activity tracking
- Use `publishEvent()` from `apps/api/src/events/`
- Events tracked for features like status changes, assignments, etc.

## Code Style

### Formatting (Biome)
- **Indentation**: Spaces for JavaScript/TypeScript/TSX (tabs for other file types)
- **Quotes**: Double quotes
- **Semicolons**: Required
- **Ignored files**: CSS and `package.json` files are excluded from Biome linting/formatting
- Run `pnpm lint` to auto-fix

### TypeScript Conventions
- Prefer `type` over `interface` (only use interface when extending/merging)
- Prefer type inference when obvious
- File naming: PascalCase for components, kebab-case for utilities/hooks
- Hooks use `use` prefix: `use-task.ts`

### Import Organization
1. External packages
2. Internal packages (`@/` aliases)
3. Relative imports
Biome auto-organizes imports.

### Git Commits
Use Conventional Commits:
- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation
- `refactor:` - Code refactoring
- `chore:` - Maintenance tasks

Husky enforces commit message format via commitlint.

### Pre-commit Hooks
The pre-commit hook (`.husky/pre-commit`) runs two checks:
1. `biome ci .` — linting and formatting validation
2. `pnpm run build` — full monorepo build

Commits will be slow due to the build step. Ensure code compiles before committing.

## Environment Configuration

**Single `.env` file** in project root shared by all apps.

Required variables:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SamalehZen/HyperFix-Kanban](https://github.com/SamalehZen/HyperFix-Kanban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
