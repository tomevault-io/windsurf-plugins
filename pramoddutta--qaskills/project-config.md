---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

QASkills.sh is a QA skills directory for AI coding agents. It's a pnpm monorepo (pnpm 9.15.0, Node >= 20) using Turborepo for orchestration.

## Common Commands

```bash
pnpm install                  # Install all dependencies
pnpm build                    # Build all packages (Turbo, respects dependency graph)
pnpm dev                      # Dev servers for all packages
pnpm test                     # Run tests across all packages (vitest)
pnpm lint                     # Lint all packages
pnpm format                   # Format with Prettier
pnpm format:check             # Check formatting

# Run a single package
pnpm --filter @qaskills/cli test
pnpm --filter @qaskills/web dev
pnpm --filter @qaskills/shared build

# Run tests in watch mode
pnpm --filter @qaskills/cli test:watch

# Database (proxied to @qaskills/web)
pnpm db:push                  # Push schema to Neon Postgres (dev)
pnpm db:migrate               # Run Drizzle migrations (prod)
pnpm db:seed                  # Seed 20 initial skills (tsx src/db/seed.ts)
pnpm db:studio                # Open Drizzle Studio

# CLI testing after build
node packages/cli/dist/index.js <command>
```

**Build order matters:** `@qaskills/shared` must build before packages that depend on it (cli, sdk, web, skill-validator). Turbo handles this via `"dependsOn": ["^build"]`. After changing shared, rebuild it before restarting dependent dev servers.

## Monorepo Layout

| Package | Path | Purpose |
|---|---|---|
| `@qaskills/shared` | `packages/shared` | Types, constants, Zod schemas, SKILL.md parser (private, dependency of all others) |
| `@qaskills/cli` | `packages/cli` | CLI tool — `qaskills add/search/init/list/remove/publish` (Commander.js + @clack/prompts) |
| `@qaskills/sdk` | `packages/sdk` | Programmatic TypeScript SDK |
| `@qaskills/skill-validator` | `packages/skill-validator` | Validates SKILL.md files against schema |
| `@qaskills/web` | `packages/web` | Next.js 15 App Router dashboard + API |
| (standalone) | `landingpage/` | Separate Next.js 16 marketing site (**not** in pnpm workspace) |
| (data) | `seed-skills/` | 20 seed QA skill definitions as SKILL.md files |

## Architecture

### Shared Package (`packages/shared`)
The single source of truth for the type system. All other packages depend on it.
- **Types:** `src/types/` — Skill, SkillSummary, SkillFrontmatter, Agent, User, Review, Category, SkillPack
- **Constants:** `src/constants/` — 30+ AgentDefinition objects (each with `configDir`, `skillsDir`, `configFile`, `installMethod`), testing frameworks, languages, domains, testing types
- **Schemas:** `src/schemas/` — Zod validation for skill frontmatter, skill creation, search params
- **Parsers:** `src/parsers/skill-parser.ts` — `parseSkillMd()` / `serializeSkillMd()` using gray-matter for YAML frontmatter + markdown body
- **Utils:** `src/utils/` — Slug generation, quality score calculation

### Web App (`packages/web`)
Next.js 15 with App Router, React 19, TailwindCSS v4, shadcn/ui (Radix primitives).

**Stack:** Neon Postgres (Drizzle ORM) / Typesense (search) / Upstash Redis (cache) / Clerk (auth) / PostHog (analytics) / Resend (email)

#### Lazy Initialization Pattern
Both the database and Resend email client use lazy singletons to avoid build-time errors on Vercel:
- **Database** (`src/db/index.ts`): `db` is a Proxy that calls `getDb()` on first property access — connection created at runtime, not import time
- **Resend** (`src/lib/email/client.ts`): `getResendClient()` lazy singleton, exported via property getters for backward compat

#### Auth Pattern
- **Middleware** (`src/middleware.ts`): Clerk middleware protects routes via `createRouteMatcher`
- **Protected routes:** `/dashboard(.*)`, `/api/skills/create(.*)`, `/api/reviews(.*)`
- **Public webhooks:** `/api/webhooks(.*)` bypasses auth entirely
- **API auth helper** (`src/lib/api-auth.ts`): `getAuthUser()` fetches the authenticated Clerk user, then looks up/auto-creates the DB row if missing (handles missed webhooks via `onConflictDoNothing`)

#### Server/Client Component Split (Key Pattern)
This is a critical pattern used throughout the app to avoid RSC errors:
- **Server components** fetch data and pass serializable props to client components
- **Client components** (`'use client'`) handle interactivity, `useAuth()`, `onClick`, PostHog tracking
- **Icons as string keys**: When server components render lists that need client interactivity, icon names are passed as strings and mapped to Lucide components in the client component (e.g., `FilterTabs`, `PacksGrid`)
- **SignupGate** (`src/components/auth/signup-gate.tsx`): Client component wrapping content behind Clerk login — used for packs gating, install buttons

**DO NOT add `onClick` or other event handlers in server components — this causes 500 errors in production.**

#### Markdown Rendering

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PramodDutta/qaskills](https://github.com/PramodDutta/qaskills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
