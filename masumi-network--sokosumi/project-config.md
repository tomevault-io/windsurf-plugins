---
trigger: always_on
description: > **Purpose**: This document provides comprehensive guidelines for AI agents working on the Sokosumi monorepo. For app-specific details, see [`apps/web/AGENTS.md`](./apps/web/AGENTS.md), [`apps/core/AGENTS.md`](./apps/core/AGENTS.md), [`packages/database/AGENTS.md`](./packages/database/AGENTS.md), and [`packages/masumi/AGENTS.md`](./packages/masumi/AGENTS.md).
---

# Sokosumi Agent Guidelines

> **Purpose**: This document provides comprehensive guidelines for AI agents working on the Sokosumi monorepo. For app-specific details, see [`apps/web/AGENTS.md`](./apps/web/AGENTS.md), [`apps/core/AGENTS.md`](./apps/core/AGENTS.md), [`packages/database/AGENTS.md`](./packages/database/AGENTS.md), and [`packages/masumi/AGENTS.md`](./packages/masumi/AGENTS.md).

## Tech Stack & Architecture

**Core Stack**: Next.js 16 (App Router), React 19.2, TypeScript, pnpm workspace, Node.js 24.x  
**Web Architecture**: Three-layer pattern with repositories (`@sokosumi/database`) wrapping Prisma/Postgres, services (`src/lib/services/`) coordinating domain flows, and actions (`src/lib/actions/`) exposing typed server mutations
**API Architecture**: Hono with OpenAPI validation and standardized response helpers
**Styling**: Tailwind CSS + shadcn/ui + Radix UI primitives
**Auth**: Better Auth with organization-aware sessions
**i18n**: next-intl for internationalization

## Project Layout

```
sokosumi/
├── apps/
│   ├── web/                   # Next.js web application
│   │   ├── src/app/           # App Router routes, server actions, API handlers
│   │   ├── src/components/    # Shared UI components
│   │   ├── src/hooks/         # Custom React hooks
│   │   ├── src/contexts/      # React contexts
│   │   ├── src/lib/           # Domain logic (services, actions, utilities)
│   │   │   ├── services/      # Business logic coordination
│   │   │   ├── actions/       # Server mutations
│   │   │   └── utils/         # Helper functions and transformers
│   │   ├── __tests__/         # Colocated tests
│   │   ├── __mocks__/         # Reusable test doubles
│   │   ├── public/            # Static assets
│   │   └── messages/          # Translation catalogs
│   └── core/                  # Hono API service
│       ├── src/routes/v1/     # API route handlers (versioned)
│       ├── src/middleware/    # Request middleware (auth, etc.)
│       ├── src/helpers/       # Response and error helpers
│       ├── src/schemas/       # Zod validation schemas
│       └── src/lib/           # Shared utilities
├── packages/
│   ├── database/              # Shared database layer (@sokosumi/database)
│   │   ├── src/repositories/  # Prisma/Postgres access layer
│   │   ├── src/helpers/       # Database domain logic
│   │   ├── src/types/         # Database type definitions
│   │   └── prisma/            # Database schema and migrations
│   ├── masumi/                # Masumi protocol utilities (@sokosumi/masumi)
│   │   ├── src/clients/       # Agent API client
│   │   ├── src/hash/          # Hash utilities for job verification
│   │   ├── src/schemas/       # Agent protocol Zod schemas
│   │   └── src/types/         # Agent types
│   └── utils/                # Shared utilities (@sokosumi/utils)
│       └── src/               # URL/file helpers, markdown link extraction, user-name, etc.
├── docs/                      # Documentation (future)
└── biome.jsonc                # Root Biome configuration
```

## Authoritative Conventions

### UI & Styling

- **Components**: Use Shadcn UI and Radix UI primitives
- **Styling**: Tailwind CSS with responsive design
- **Colors**: Use semantic colors from `globals.css`; never hardcode hex values
- **Sizing**: Use `size-4` instead of `h-4 w-4`
- **Themes**: Ensure compatibility with both dark and light modes

### TypeScript Usage

- **Mandatory**: Use TypeScript for all code
- **Interfaces**: Prefer interfaces over types
- **Enums**: Avoid enums; use maps instead
- **Components**: Use functional components with TypeScript interfaces
- **Inference**: Leverage Prisma type inference when possible

### Key Conventions

- **URL State**: Use `nuqs` for URL search parameter state management
- **Client Components**: Limit `'use client'` usage
  - Favor server components and Next.js SSR
  - Use only for Web API access in small components
  - Avoid for data fetching or state management
- **Async Operations**: Use Suspense for async operations
- **Data Fetching**: Follow Next.js docs for Data Fetching, Rendering, and Routing

### Naming & Patterns

- **Components**: PascalCase (e.g., `UserProfile`)
- **Types/Interfaces**: PascalCase (e.g., `UserData`)
- **Functions**: camelCase (e.g., `getUserData`)
- **Constants**: SCREAMING_SNAKE_CASE (e.g., `API_BASE_URL`)
- **Directories**: kebab-case (e.g., `user-profile`)
- **Prisma Models**: Singular (e.g., `User`, not `Users`)
- **Event Handlers**: Prefix with `handle` (e.g., `handleSubmit`)
- **Exports**: Prefer named exports
- **Functions**: Use `function` keyword for pure functions

### Code Style

- **Indentation**: Two spaces, semicolons enforced by Biome
- **Formatting**: Run `pnpm format` after substantial edits
- **Imports**: Relative within features, use aliases (`@/lib/*`) otherwise
- **Components**: Default to Server Components; add `'use client'` only for browser APIs

### Linting & Formatting

#### Biome Configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masumi-network/sokosumi](https://github.com/masumi-network/sokosumi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
