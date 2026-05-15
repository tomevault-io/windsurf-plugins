---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI SaaS Template - A production-ready, enterprise-grade full-stack TypeScript template for building AI-powered SaaS applications. Built with Next.js 16 App Router, React 19.2, tRPC 11, Drizzle ORM, and Better Auth.

**Key Stats**: ~43k lines of code, 49+ pages/layouts, fully internationalized (zh/en)

## Essential Commands

### Development
```bash
pnpm dev                    # Start dev server (Turbo mode)
pnpm build                  # Production build
pnpm start                  # Start production server
```

### Code Quality
```bash
pnpm lint                   # Biome check + auto-fix
pnpm type-check             # TypeScript validation
pnpm ci                     # Full CI check (type + lint + test)
pnpm quality:check          # Complete quality audit (type + lint + format + coverage)
```

### Testing
```bash
pnpm test                   # Run all tests
pnpm test:dev               # Watch mode
pnpm test:coverage          # Generate coverage report
pnpm test:e2e               # Run E2E tests (Playwright)
```

### Database
```bash
pnpm db:generate            # Generate migration files
pnpm db:migrate             # Run migrations
pnpm db:studio              # Open Drizzle Studio GUI
pnpm db:push                # Push schema to database (dev only)
```

### Single Test Execution
```bash
pnpm test -- path/to/test.test.ts           # Run specific test file
pnpm test -- -t "test name pattern"         # Run tests matching pattern
```

## Architecture Overview

### Tech Stack
- **Frontend**: Next.js 16 (App Router + Cache Components), React 19.2, TypeScript 6 (strict mode)
- **API Layer**: tRPC 11.16 (end-to-end type safety)
- **Database**: PostgreSQL + Drizzle ORM 0.45
- **Auth**: Better Auth 1.6 (email/password + Google OAuth + GitHub OAuth, RBAC, email verification, login security)
- **Payments**: Stripe 22 (subscriptions + one-time payments + webhooks + coupons + refunds)
- **AI**: Vercel AI SDK 6 (OpenAI, Anthropic, Google AI, xAI) with multi-model switching, token tracking, quota control, RAG, Agent workflows
- **UI**: Tailwind CSS v4.2 + shadcn/ui (Radix primitives)
- **State**: TanStack Query 5 + React Context
- **i18n**: next-intl 4.9 (zh/en server + client translations)
- **Code Quality**: Biome 2.4 (replaces ESLint/Prettier), Vitest 4, Playwright
- **Caching**: Redis (Upstash) for rate limiting and performance
- **Observability**: Structured logging (JSON), Sentry error tracking, tRPC performance tracing
- **Docs**: Fumadocs 16 documentation system

### Layered Architecture Pattern

```
Presentation Layer (Next.js Pages + React Components)
       ↓
Business Logic Layer (Custom Hooks + Context Providers)
       ↓
API Gateway Layer (tRPC Routers + Middleware)
       ↓
Service Layer (Auth, Payment, AI, Email Services)
       ↓
Data Access Layer (Drizzle ORM + Schemas)
       ↓
Infrastructure (PostgreSQL, Redis, External APIs)
```

### Type Safety Chain
Database Schema (Drizzle) → tRPC Procedures → React Hooks → UI Components

### Critical Architecture Patterns

**1. tRPC Procedure Hierarchy**
```typescript
publicProcedure        // No auth required
  ↓
protectedProcedure     // Authenticated users only (checks session)
  ↓
adminProcedure         // adminLevel >= 1
  ↓
superAdminProcedure    // adminLevel >= 2
```
- Defined in: `src/server/server.ts`
- All business logic MUST go through tRPC procedures, not direct DB calls in components

**2. Authentication System (Better Auth)**
- Location: `src/lib/auth/better-auth/`
- Flow: Better Auth → tRPC Context → Middleware → Protected Routes
- Admin levels: 0=user, 1=admin, 2=super admin
- CRITICAL: `BETTER_AUTH_SECRET` requires 32+ characters
- Preferences stored as JSON strings in database (must parse/stringify)

**3. Database Layer (Drizzle ORM)**
- Schemas: `src/drizzle/schemas/` (organized by domain)
- Relations: `src/drizzle/schemas/relations.ts`
- Migrations: `src/drizzle/migrations/`
- ALWAYS use Drizzle queries, never raw SQL
- Schema changes require: modify schema → `pnpm db:generate` → `pnpm db:migrate`

**4. Component Conventions**
- Server Components by default (no 'use client')
- Client Components only when needed (state, hooks, browser APIs)
- shadcn/ui components in `src/components/ui/`
- Import aliases: `@/*` → `src/*`

**5. Internationalization**
```typescript
// Server Components
import { getTranslations } from 'next-intl/server'
const t = await getTranslations('namespace')

// Client Components
'use client'
import { useTranslations } from 'next-intl'
const t = useTranslations('namespace')
```
- Translation files: `src/translate/messages/zh.json` and `en.json`
- ALL user-facing text must be internationalized

**6. Feature-Based Organization**
- Components grouped by domain (auth, admin, ai-elements, front)
- Each domain has corresponding tRPC router in `src/server/routers/`
- Database schemas organized by business domain

## Key Directories

### Core Application Structure
```
src/
├── app/[locale]/              # Internationalized routes
│   ├── (front)/              # Public pages (marketing, landing)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geallenboy/ai-saas-template](https://github.com/geallenboy/ai-saas-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
