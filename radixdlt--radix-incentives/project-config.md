---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Radix Incentives Project

This is a Turborepo monorepo for the Radix Incentives Campaign - a blockchain incentive system designed to enhance significant and sustained on-chain economic activities on the Radix DLT network. The platform tracks user activities across DeFi protocols, calculates incentive points using time-weighted averages, and provides dashboards for both end users and administrators.

## Architecture Overview

### Applications
- **`apps/admin`** - Next.js admin dashboard for managing seasons, weeks, activities, and viewing analytics
- **`apps/incentives`** - Next.js user-facing dashboard where users connect wallets, view points, leaderboards, and activities  
- **`apps/workers`** - Background job processors using Bull queues for calculating points, snapshots, and processing events
- **`apps/streamer`** - Transaction stream processor that monitors Radix blockchain for relevant events

### Packages
- **`packages/api`** - Shared API layer containing business logic for all applications
- **`packages/db`** - Drizzle ORM database schemas and migrations for both incentives and consultation systems
- **`packages/data`** - Shared type definitions, constants, and validation schemas

### Key Data Flow
1. **Transaction Stream** → Events are captured from Radix blockchain via gateway API
2. **Event Processing** → Background workers match and process events for specific DApps (Ociswap, DefiPlaza, CaviarNine, etc.)
3. **Snapshot System** → Periodic account balance snapshots are taken for passive activities
4. **Points Calculation** → Activity points are calculated based on time-weighted averages and user actions
5. **Season Points** → Activity points are aggregated into season points with XRD/LSU holding multipliers

## Development Commands

### Environment Setup
```bash
# Install dependencies (use pnpm, not npm)
pnpm install

# Start database (requires Docker)
pnpm db:start

# Set required environment variables
export DATABASE_URL="postgres://postgres:password@localhost:5432/radix-incentives"

# Run database migrations
pnpm db:migrate

# Seed database with initial data
cd packages/db && pnpm db:seed
```

### Development
```bash
# Run all apps in development
pnpm dev

# Run specific applications
pnpm dev:admin      # Admin dashboard + db + workers
pnpm dev:fe         # User incentives app + db  
pnpm dev:workers    # Workers + db + streamer
pnpm dev:streamer   # Streamer + db
```

### Database Operations
```bash
# Generate new migrations after schema changes
pnpm db:generate

# Apply migrations
pnpm db:migrate

# Launch Drizzle Studio
pnpm db:studio

# Reset database (drops all data)
pnpm db:reset
```

### Code Quality
```bash
# Format code with Biome
pnpm format

# Lint and fix issues
pnpm biome lint --write

# Type check all packages
pnpm check-types

# Run tests (DATABASE_URL must be set)
DATABASE_URL="postgresql://postgres:password@localhost:5432/radix-incentives" pnpm test
```

#### Lint & Build Process
When fixing lint/build issues, follow this order:
1. **Run lint**: `pnpm biome lint` - fix auto-fixable issues with `--write`
2. **Manual fixes required**:
   - Replace `forEach` with `for...of` loops for performance
   - Add `type` to button elements: `<button type="button">`
   - Use `import type` for React imports: `import type * as React from "react"`
   - Add keyboard event handlers for click events (accessibility)

3. **Run build**: `pnpm build` - clean with `pnpm clean` if workspace conflicts occur
4. **Commit**: All changes including lint fixes, build fixes, and accessibility improvements

**Biome Configuration**: 
- Ignores `.next`, `**/output/**` directories to prevent linting build artifacts
- Auto-fixes import types and other style issues with `--write` flag

### Build & Deploy
```bash
# Build all applications
pnpm build

# Build with cleanup
pnpm build:clean
```

### Troubleshooting
```bash
# Fix Turborepo workspace conflicts (e.g., "Failed to add workspace, it already exists")
# This happens when Next.js standalone builds create duplicate package.json files
pnpm clean
```

## Core Development Principles

### Context and Rules
- **Incremental Changes**: Make changes file by file to allow for review
- **No Assumptions**: Do not invent changes, make assumptions, or speculate without evidence from the context

### Technology Stack Specifics
- **Frontend**: Next.js, ReactJS, TypeScript, TailwindCSS, Shadcn, Radix UI
- **Backend**: Node.js, TypeScript, tRPC v11
- **Database**: PostgreSQL with Drizzle ORM
- **Caching**: Redis
- **Job Processing**: Bull MQ
- **Blockchain**: Radix Gateway SDK, Radix-dApp-toolkit

### TypeScript Guidelines
- Use `type` over `interface`
- Use functions over classes
- Use named exports over default exports
- Use `const` arrow functions with types
- Document all functions
- Write Vitest unit tests covering all functions
- Use the `Effect` library for functional composition (`pipe`)
- Import types: `import type { FC } from "react"` not `import { type FC } from "react"`

### Effect.Service Guidelines
When creating services using the Effect library:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radixdlt/radix-incentives](https://github.com/radixdlt/radix-incentives) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
