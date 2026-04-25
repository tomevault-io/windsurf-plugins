---
trigger: always_on
description: EventLayer is a full-service event management platform built as a monorepo using **pnpm workspaces** and **Turborepo**. It enables organizers to create and manage events with features like attendee management, scheduling, ticketing, venue mapping, sponsor management, and a PWA-based mobile app experience.
---

# CLAUDE.md - AI Assistant Guide for EventLayer

## Project Overview

EventLayer is a full-service event management platform built as a monorepo using **pnpm workspaces** and **Turborepo**. It enables organizers to create and manage events with features like attendee management, scheduling, ticketing, venue mapping, sponsor management, and a PWA-based mobile app experience.

## Tech Stack

- **Package Manager**: pnpm (v9.1.1)
- **Monorepo Tool**: Turborepo
- **Frontend**: SvelteKit 2.x with Svelte 4.x
- **Styling**: TailwindCSS with PostCSS
- **Database**: PostgreSQL with Drizzle ORM
- **API**: tRPC for type-safe API calls
- **Authentication**: Lucia Auth
- **Caching**: Redis (ioredis/Upstash)
- **Email**: Resend
- **Payments**: Stripe
- **File Storage**: AWS S3
- **Testing**: Vitest (unit) + Playwright (e2e)
- **Language**: TypeScript (strict mode)

## Repository Structure

```
eventlayer/
├── apps/
│   └── web/                    # Main SvelteKit application
│       ├── src/
│       │   ├── routes/         # SvelteKit file-based routing
│       │   │   ├── (api)/      # API endpoints (Stripe webhooks, etc.)
│       │   │   ├── (checkin)/  # Event check-in functionality
│       │   │   ├── (manage)/   # Admin/staff management dashboard
│       │   │   └── (pwa)/      # Main PWA app for attendees
│       │   ├── lib/
│       │   │   ├── components/ # Svelte components
│       │   │   ├── server/     # Server-side utilities
│       │   │   └── state/      # State management
│       │   ├── hooks.server.ts # Server hooks (auth, CORS, tRPC)
│       │   └── hooks.client.ts # Client hooks
│       ├── static/             # Static assets
│       └── tests/              # Playwright tests
├── packages/
│   ├── api/                    # tRPC router and API logic
│   │   └── src/
│   │       ├── router/         # tRPC procedures by domain
│   │       ├── models/         # Business logic functions
│   │       ├── core/           # Auth, Redis, errors
│   │       └── media/          # Media handling (S3 uploads)
│   ├── db/                     # Database schema and configuration
│   │   └── schema/             # Drizzle schema files by table
│   ├── ui/                     # Shared UI component library
│   │   └── src/components/     # Reusable Svelte components
│   └── util/                   # Shared utilities (lodash wrappers, dayjs, etc.)
└── tooling/
    ├── eslint/                 # Shared ESLint configuration
    ├── prettier/               # Shared Prettier configuration
    ├── tailwind/               # Shared Tailwind configuration
    └── typescript/             # Shared TypeScript configuration
```

## Package Naming Convention

All internal packages use the `@matterloop/` scope:
- `@matterloop/api` - API/tRPC package
- `@matterloop/db` - Database package
- `@matterloop/ui` - UI components
- `@matterloop/util` - Utilities
- `@matterloop/eslint-config` - ESLint config
- `@matterloop/prettier-config` - Prettier config
- `@matterloop/tsconfig` - TypeScript config

## Development Commands

```bash
# Install dependencies
pnpm install

# Start development server (web app only)
pnpm dev

# Build all packages
pnpm build

# Run linting
pnpm lint

# Format code
pnpm format

# Database operations (from packages/db or apps/web)
pnpm db:push        # Push schema changes to database
pnpm db:gen         # Generate migrations
pnpm db:introspect  # Introspect existing database

# Testing (from apps/web)
pnpm test              # Run all tests
pnpm test:unit         # Run Vitest unit tests
pnpm test:integration  # Run Playwright tests
```

## Code Conventions

### TypeScript

- Strict mode enabled (`noUncheckedIndexedAccess: true`)
- Use type inference where possible
- Export types alongside implementations
- Use Zod schemas for runtime validation

### Svelte Components

- Use `lang="ts"` in script blocks
- Follow component naming: `PascalCase.svelte`
- Use `$lib/` alias for imports from `src/lib/`
- Context API for global state (`setContext`/`getContext`)
- Svelte stores with `writable`/`persisted`

### Styling

- TailwindCSS for styling (utility-first)
- Use `tw()` utility from `@matterloop/ui` for conditional classes
- PostCSS for processing
- Dark mode via class strategy
- CSS variables for theming (HSL colors)

### Prettier Configuration

- No semicolons
- Tabs for indentation
- Single quotes
- Trailing commas
- 100 character print width
- Import sorting with `@ianvs/prettier-plugin-sort-imports`

Import order:
1. React/Next (if applicable)
2. Third-party modules
3. `@matterloop/*` packages
4. Relative imports

### File Structure Patterns

**SvelteKit Routes:**
```
routes/
├── +page.svelte          # Page component
├── +page.server.ts       # Server load function
├── +layout.svelte        # Layout component
├── +layout.server.ts     # Layout server load
└── +server.ts            # API endpoint
```

**tRPC Procedures:**
```typescript
// packages/api/src/router/entityProcedures.ts
export const entityProcedures = t.router({

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nickyhajal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
