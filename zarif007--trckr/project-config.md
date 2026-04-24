---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Engineering Standard

Build every piece of this project — components, functions, features, APIs — to production grade. Not MVP. Not "good enough for now."

**The bar:** Code a new engineer (or agent) can onboard into in minutes, not days. Self-explanatory names, clear boundaries, no mystery.

### Non-negotiables

- **Modular**: Every unit has one job. Extract reusable logic into `lib/`. Keep components focused on rendering/interaction only.
- **Scalable**: Design for growth. Avoid patterns that require rewriting when requirements change (e.g. hardcoded lists, prop-drilling across 3+ levels, god components).
- **Unbreakable**: Cover edge cases. Validate at system boundaries. Never silently swallow errors. Tests for any non-trivial logic.
- **Self-documenting**: Names explain intent. A function named `resolveFieldVisibility` needs no comment. Avoid abbreviations. Prefer explicit over clever.
- **Minimal comments**: Only comment *why*, never *what*. If the code needs explanation, rename or restructure first.
- **Consistent**: Follow existing patterns in the codebase before inventing new ones. New patterns belong in `lib/` with clear ownership.

### What this means in practice

- New feature? Define its module boundary before writing code.
- New component? It should be droppable into a different page without changes.
- New lib function? Export a typed interface, handle error cases, write a test.
- Refactoring? Leave the module more coherent than you found it — but only touch what's necessary for the task.
- If you find a shortcut that works now but will break at scale, flag it and do it right.

## Commands

```bash
npm run dev              # Start development server (localhost:3000)
npm run build            # Production build (runs prisma generate first)
npm run lint             # Run ESLint
npm run typecheck        # TypeScript type checking
npm run test             # Run vitest in watch mode
npm run test:run         # Run all tests once
vitest run <path>        # Run specific test file

# Database
npm run db:generate      # Generate Prisma client
npm run db:migrate       # Run migrations (dev)
npm run db:push          # Push schema to database
npm run db:studio        # Open Prisma Studio

# Docs
npm run docs:generate    # Regenerate file/route/module maps
npm run docs:check       # Validate docs coverage and sync

# Quality gates (required before merge)
npm run lint && npm run test:run && npm run typecheck && npm run docs:check
```

## Architecture

### Core Data Flow

Trckr is a Next.js App Router app for AI-assisted tracker generation. Users create trackers via chat, and the system renders them as interactive grids (table, kanban, form views) with bindings and conditional rules.

1. **Schema Flow**: Tracker schema loads from API → `TrackerAIView` coordinates AI generation → `TrackerDisplay` renders schema and emits updates → Save persists via PATCH
2. **Data Flow**: Snapshot fetched → passed to `TrackerDisplay` as grid data → user edits → Save persists snapshot
3. **Runtime Engines**: Grid rendering uses `lib/binding` (option resolution), `lib/field-rules` (visibility/required/disabled), `lib/field-validation`, `lib/field-calculation`, `lib/dynamic-options`

### Directory Structure

- `app/` — Routes and feature UI
  - `app/components/tracker-display/` — Schema runtime renderer/editor (tabs, sections, grids, field settings)
  - `app/components/tracker-page/` — Tracker chat and page-level presentation
  - `app/tracker/` — Tracker route orchestration and chat/generation flow
  - `app/api/` — HTTP boundary, auth checks, request/response validation
- `lib/` — Domain services and reusable runtime logic
  - `lib/repositories/` — Persistence + ownership checks for API routes
  - `lib/auth/server/` — Auth handler boundary
  - `lib/ai/` — Provider abstraction, AI config, timeout/logging
  - `lib/binding/`, `lib/field-rules/`, `lib/validate-tracker/` — Core domain engines
- `components/` — Shared UI primitives (shadcn-based)
- `prisma/` — Database schema (PostgreSQL)

### Module Boundaries

- UI (`app/components`) depends on `lib/*`
- API handlers depend on `lib/*` and DB layer
- `lib/*` modules do NOT depend on route components
- Feature orchestration stays near routes; cross-feature domain logic goes in `lib/*`

### API Handler Pattern

```typescript
// 1. Auth
const { userId } = await requireAuthenticatedUser()
// 2. Params/body validation with lib/api helpers + Zod
// 3. Repository/service calls
// 4. Return jsonOk / jsonError from lib/api/http
```

### Prompt-Building Functions (Agent & AI Flows)

**Critical rule**: All variables used in template literals must be function parameters.

When building prompts for AI agents/tools, prompt functions receive context data. **Every variable referenced in a template literal must be explicitly passed as a parameter** — never rely on outer scope or expect TypeScript to infer it.

**Anti-pattern**:
```typescript
export function buildSystemPrompt(purpose: string): string {
  return `Use gridId: ${gridId}` // ❌ gridId undefined — not a parameter
}
```

**Correct pattern**:
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zarif007) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
