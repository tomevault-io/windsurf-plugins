---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MUNify DELEGATOR is a SvelteKit-based application for managing Model United Nations conference registration, delegation assignment, and organizational matters. Built with Svelte 5, TypeScript, Prisma ORM, and GraphQL (via Pothos & Yoga), it's designed for DMUN e.V. but can be adapted for other MUN conferences.

## Tech Stack Core

- **Frontend**: SvelteKit with Svelte 5 (runes mode), TailwindCSS 4, DaisyUI
- **Backend**: Node.js with SvelteKit server routes
- **Database**: PostgreSQL via Prisma ORM
- **GraphQL**: Pothos schema builder with graphql-yoga server, Houdini client
- **Auth**: OpenID Connect (OIDC) - recommended provider: Logto
- **i18n**: Paraglide-JS for internationalization (default locale: German)
- **Runtime**: Bun (package manager and development runtime)
- **Observability**: OpenTelemetry tracing support

## Development Commands

### Setup & Running

```bash
# Install dependencies
bun install

# Start development (Docker + dev server)
bun run dev

# Separate commands
bun run dev:docker    # Start Docker containers (PostgreSQL, etc.)
bun run dev:server    # Start Vite dev server only

# Install git hooks (lefthook)
bunx lefthook install
```

### Database Operations

```bash
# Generate Prisma client
bunx prisma generate

# Create and apply migrations
bunx prisma migrate dev

# Reset database (WARNING: deletes all data)
bunx prisma migrate reset

# Seed database with development data
bun prisma/seed/dev/seed.ts

# Open Prisma Studio (database GUI)
bun run studio
```

### Code Quality

```bash
# Format code
bun run format

# Type checking (PREFERRED for development - fast feedback)
bun run check          # Single run - use this to verify changes
bun run check:watch    # Watch mode
bun run typecheck      # TypeScript only

# Lint (slow - runs automatically on git push via lefthook)
bun run lint           # Only run manually when specifically needed

# Testing
bun test               # Run tests once
bun run test:watch     # Watch mode
bun run coverage       # With coverage report
```

**Note:** Prefer `bun run check` over `bun run lint` during development. Linting is slow (~2 min) and runs automatically on push via lefthook pre-push hooks. Use `bun run check` for quick type-checking feedback.

### i18n (Internationalization)

```bash
# Check for missing/unused translations
bun run i18n:check

# Validate translation project
bun run i18n:validate

# Auto-translate missing keys
bun run i18n:translate

# Add new translation key interactively
bun run add-translation
```

### Building

```bash
# Production build
bun run build

# Preview production build
bun run preview
```

## Architecture

### Directory Structure

- **`src/routes/`** - SvelteKit routes (filesystem-based routing)
  - `(authenticated)/` - Protected routes requiring authentication
    - `dashboard/` - Conference dashboards
    - `management/` - Admin interfaces for conference/delegation management
    - `registration/` - User registration flows
    - `assignment-assistant/` - Delegation assignment tools
  - `api/graphql/` - GraphQL API endpoint
  - `auth/` - OIDC authentication callbacks
  - `seats/`, `vc/`, `validateCertificate/` - Public-facing pages

- **`src/api/`** - Backend GraphQL API layer
  - `resolvers/` - GraphQL resolvers organized by module
    - `modules/` - Domain-specific resolvers (conference, delegation, committee, etc.)
    - `api.ts` - Main GraphQL Yoga server configuration
    - `builder.ts` - Pothos schema builder setup with plugins
  - `abilities/` - CASL-based authorization rules per entity
  - `services/` - Backend business logic services

- **`src/lib/`** - Shared frontend code
  - `components/` - Reusable Svelte components
  - `queries/` - Houdini GraphQL queries/mutations
  - `services/` - Frontend utility functions
  - `schemata/` - Zod validation schemas
  - `paraglide/` - Generated i18n code

- **`src/config/`** - Environment configuration
  - `public.ts` - Client-side config (PUBLIC\_\* env vars)
  - `private.ts` - Server-side config (secrets, DB URLs)

- **`src/tasks/`** - Background scheduled tasks (node-schedule)
  - `conferenceStatus.ts` - Auto-update conference states
  - `mailSync.ts` - Email synchronization with external systems

- **`prisma/`** - Database layer
  - `schema.prisma` - Prisma schema definition
  - `migrations/` - Database migration history
  - `seed/` - Database seeding scripts
  - `pothos/` - Generated Pothos types from Prisma

### Key Architectural Patterns

#### 1. GraphQL API Layer (Pothos + Yoga)

- **Schema-first approach** using Pothos code-first schema builder
- All resolvers in `src/api/resolvers/modules/` are auto-imported in `api.ts`
- Plugin stack: Prisma integration, complexity limiting, OpenTelemetry tracing, utilities
- **Authorization**: CASL ability checks per entity type in `src/api/abilities/entities/`
- Context includes: Prisma client, OIDC user info, request metadata

#### 2. Frontend Data Fetching (Houdini)

- **Houdini** manages GraphQL client-side operations
- Type-safe generated queries in `.houdini/` directory
- Queries/mutations in `src/lib/queries/` trigger codegen

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DeutscheModelUnitedNations) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
