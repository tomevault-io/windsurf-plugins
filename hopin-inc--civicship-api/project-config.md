---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**civicship-api** is a GraphQL API server built with TypeScript, following **Domain-Driven Design (DDD)** and **Clean Architecture** principles. It provides a platform for community engagement with point-based rewards, opportunity management, and LINE messaging integration.

**Tech Stack:** TypeScript + Node.js + Apollo Server + Prisma ORM + PostgreSQL + Firebase Auth

## Essential Commands

### Development
```bash
# Start development server (HTTPS)
pnpm dev:https

# Start development server (HTTP)
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start
```

### Database Operations
```bash
# Generate Prisma client and types
pnpm db:generate

# Create new migration (after schema changes)
pnpm db:migrate

# Apply migrations to database
pnpm db:deploy

# Seed master data (cities, states)
pnpm db:seed-master

# Seed domain data (users, communities)
pnpm db:seed-domain

# Open Prisma Studio (database GUI)
pnpm db:studio

# Pull schema from database
pnpm db:pull
```

### GraphQL Code Generation
```bash
# Generate TypeScript types from GraphQL schemas
pnpm gql:generate
```

### Testing
```bash
# Run all tests (serial execution with --runInBand)
pnpm test

# Run tests with coverage
pnpm test:coverage

# Run specific test file
pnpm test path/to/test.test.ts

# Run tests in watch mode
pnpm test --watch
```

### Code Quality
```bash
# Run linter and formatter
pnpm lint

# Format code only
prettier --write src/
```

### Docker
```bash
# Start PostgreSQL container (port 15432)
pnpm container:up

# Stop and remove containers
pnpm container:down
```

## Architecture Overview

### Layer Structure

The codebase follows **Clean Architecture** with 3 main layers:

1. **Application Layer** (`src/application/`) - Business logic and domain operations
2. **Infrastructure Layer** (`src/infrastructure/`) - Database, external services, Firebase, GCS
3. **Presentation Layer** (`src/presentation/`) - GraphQL API, middleware, REST endpoints

### Core Business Domains

Located in `src/application/domain/`:

- **account/** - User, Community, Membership, Identity, Wallet, NFT Wallet (8 subdomains)
- **experience/** - Opportunity, Reservation, Participation, Evaluation
- **content/** - Article, Image management
- **reward/** - Utility, Ticket, point-based rewards
- **transaction/** - Point transfers between wallets
- **notification/** - LINE messaging integration
- **location/** - Geographic data (Place, City, State)

### Standard Domain Pattern

Each domain follows this consistent structure:

```
domain/{domain-name}/
├── controller/
│   ├── resolver.ts         # GraphQL Query/Mutation/field resolvers
│   └── dataloader.ts       # N+1 prevention (batch loading)
├── usecase.ts             # Business flow orchestration
├── service.ts             # Business logic & validation
├── data/
│   ├── repository.ts      # Prisma database queries
│   ├── interface.ts       # Repository interface contract
│   ├── converter.ts       # GraphQL input → Prisma format
│   └── type.ts            # TypeScript types (Prisma select shapes)
├── presenter.ts           # Prisma → GraphQL response formatting
└── schema/
    ├── query.graphql      # Query definitions
    ├── mutation.graphql   # Mutation definitions
    └── type.graphql       # Type definitions
```

### Data Flow Pattern

**Request Flow:**
```
GraphQL Request
  → Resolver (controller/resolver.ts)
  → UseCase (usecase.ts) - orchestration, authorization, transactions
  → Service (service.ts) - business logic, validation
  → Repository (data/repository.ts) - Prisma queries
  → Database
```

**Response Flow:**
```
Prisma Result
  → Presenter (presenter.ts) - format to GraphQL types
  → UseCase
  → Resolver
  → GraphQL Response
```

## Critical Implementation Rules

### Layer Responsibilities

**DO NOT violate these rules - they are core architectural principles:**

1. **Resolver** (`controller/resolver.ts`)
   - ✅ Call usecase methods only
   - ✅ Use dataloaders for field resolvers to prevent N+1
   - ❌ No business logic
   - ❌ No direct repository calls

2. **UseCase** (`usecase.ts`)
   - ✅ Orchestrate business flows
   - ✅ Manage transactions via `ctx.issuer.onlyBelongingCommunity(ctx, async tx => {...})`
   - ✅ Call services (own domain and other domains)
   - ✅ Call presenters to format responses
   - ❌ Never call other domain's usecases (circular dependency risk)

3. **Service** (`service.ts`)
   - ✅ Implement business logic and validation
   - ✅ Call repositories
   - ✅ Call other domain services (read operations only)
   - ✅ Handle `tx` parameter with `if (tx)` branching
   - ❌ No cross-domain usecase calls
   - ❌ Never return GraphQL types (`GqlXxx`) - return Prisma types only

4. **Repository** (`data/repository.ts`)
   - ✅ Execute Prisma queries
   - ✅ Use `ctx.issuer` for Row-Level Security (RLS)
   - ✅ Handle `tx` parameter with `if (tx)` branching
   - ❌ No business logic

5. **Converter** (`data/converter.ts`)
   - ✅ Transform GraphQL input → Prisma format
   - ✅ Pure functions only (no side effects)
   - ❌ No transaction dependency
   - ❌ No database queries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hopin-inc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-16 -->
