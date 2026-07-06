---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Structure

This is a pnpm monorepo with three main applications:

- **`apps/backend`** - Fastify-based API server with tRPC (TypeScript, Node.js)
- **`apps/frontend`** - Next.js web application (TypeScript, React)
- **`apps/mobile`** - Expo/React Native mobile app (TypeScript, React Native)

All packages use TypeScript and share common development tools configured at the root.

## Getting Started

### Initial Setup

```bash
# Install all dependencies
pnpm install
```

### Environment Variables

#### Claude specific

For non-app related environment variables search in `.claude/.env`

#### Backend

The backend requires environment variables to be configured. Copy the example file and fill in the values:

```bash
cd apps/backend
cp .env.example .env
```

Required environment variables:

- `DATABASE_URL` - PostgreSQL connection string
- `CLERK_PUBLISHABLE_KEY` - Clerk authentication public key
- `CLERK_SECRET_KEY` - Clerk authentication secret key

**Note:** The `.env` file contains secrets and should not be committed to version control. It's already in `.gitignore`.

#### Frontend (Vercel)

The frontend app is deployed to Vercel and uses Clerk for authentication. To run the app locally, you need to pull environment variables from Vercel:

```bash
# Login to Vercel (if not already logged in)
npx vercel login

# Navigate to the frontend app
cd apps/frontend

# Pull environment variables from Vercel
npx vercel env pull .env.development.local
```

This will create a `.env.development.local` file with the necessary environment variables (Clerk keys, etc.) for local development.

**Note:** The `.env.development.local` file contains secrets and should not be committed to version control. It's already in `.gitignore`.

#### Mobile

The mobile app uses Clerk for authentication. Environment variables are set via `EXPO_PUBLIC_` prefix:

```bash
cd apps/mobile
# Set EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY in your .env file
```

## Development Commands

### Running the project

```bash
# Install all dependencies
pnpm install

# Run all packages in development mode (parallel)
pnpm dev

# Run a specific package
pnpm --filter @manila/backend dev
pnpm --filter @manila/frontend dev
```

### Building

```bash
# Build all packages
pnpm build

# Build a specific package
pnpm --filter @manila/backend build
pnpm --filter @manila/frontend build
```

### Testing

```bash
# Run tests across all packages
pnpm test

# Run tests for a specific package
pnpm --filter @manila/backend test

# Run backend tests with UI (Vitest)
pnpm --filter @manila/backend test:ui
```

### Linting

```bash
# Lint all packages
pnpm lint

# Lint a specific package
pnpm --filter @manila/frontend lint
```

### Cleaning

```bash
# Remove all node_modules and build artifacts
pnpm clean

# Clean a specific package
pnpm --filter @manila/backend clean
```

## Architecture

### Backend (`apps/backend`)

The backend is a Fastify server with tRPC for type-safe API endpoints.

**Key architectural components:**

- **Server Framework**: Fastify with plugins for security (helmet, CORS, rate limiting)
- **API Layer**: tRPC routers for type-safe API endpoints
- **Database**: PostgreSQL with Drizzle ORM and pgvector for embeddings
- **Authentication**: Clerk integration via `@clerk/fastify`
- **Context**: tRPC context provides authenticated user info and database access to all procedures

**Entry point**: `src/index.ts` - Sets up Fastify server with middleware and tRPC plugin

**Database schema**: `src/db/schema.ts` - Drizzle schema definitions for users, refresh_tokens, and embeddings tables

**tRPC routers**:

- `src/trpc/router.ts` - Main router that combines all sub-routers
- `src/trpc/routers/*` - Individual route handlers (health, embeddings)
- `src/trpc/context.ts` - Context creation with Clerk auth and database access

**Environment**:

- Uses `.env` for all environments (development and production)
- Schema validation via Zod in `src/lib/env.ts`
- Build output: `dist/`
- Runtime: Node.js with ES modules (`type: "module"`)
- Dev tool: `tsx` for hot-reloading

**Local development workflow (recommended):**

For faster iteration, run PostgreSQL in Docker and the backend locally:

```bash
cd apps/backend

# 1. Start PostgreSQL database
pnpm db:start

# 2. Run backend locally (in a separate terminal)
pnpm dev

# When done:
pnpm db:stop
```

**Database commands:**

```bash
cd apps/backend

# Start/stop PostgreSQL database
pnpm db:start          # Start database in Docker (runs in background)
pnpm db:stop           # Stop database
pnpm db:logs           # View database logs
pnpm db:clean          # Remove database and volumes (fresh start)

# Database migrations and schema
pnpm db:generate       # Generate migrations from schema changes
pnpm db:migrate        # Apply migrations to database
pnpm db:push           # Push schema directly to database (development)
pnpm db:studio         # Open Drizzle Studio GUI
```

**Docker commands (production):**

Use these for production deployment (full stack with backend + database in Docker):

```bash
cd apps/backend

# Production

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omniphx/FileLlama](https://github.com/omniphx/FileLlama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
