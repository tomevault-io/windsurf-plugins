---
trigger: always_on
description: Self-hosted platform of specialized AI agents (Agents) for individuals and small groups. Each Agent has a persistent identity, expertise, memory, and tools. Agents share a single continuous session (no "new conversation"), collaborate with each other, spawn sub-Agents for tasks, and execute scheduled jobs.
---

# Hivekeep

Self-hosted platform of specialized AI agents (Agents) for individuals and small groups. Each Agent has a persistent identity, expertise, memory, and tools. Agents share a single continuous session (no "new conversation"), collaborate with each other, spawn sub-Agents for tasks, and execute scheduled jobs.

## Documentation map

Read these files **before starting any phase**. They are the source of truth.

| File | Content |
|---|---|
| `schema.md` | Complete SQLite database schema (all tables, indexes, virtual tables) |
| `api.md` | REST API contracts (request/response for every route) + SSE events |
| `sse.md` | **Real-time/SSE cheat sheet** — emit↔handle rules, the 8 recurring sync-bug traps, optimistic reconciliation, review checklist. Read before touching SSE or shared state. |
| `config.md` | All configurable values with env vars and defaults |
| `structure.md` | Project file tree, naming conventions, imports, i18n, error format |
| `prompt-system.md` | How the Agent system prompt is assembled (blocks 1-12) |
| `compacting.md` | Compacting algorithm + memory extraction pipeline |
| `queenie.md` | **Conversational onboarding** spec — the `Queenie` configurator Agent, vault-centralized secrets, secure-input tools, avatar-style customization (Phase 27) |
| `files.md` | **Files section** spec — workspace file browser/editor (tree + tabs + CodeMirror), workspace REST API + `workspace:changed` SSE, share-to-file-storage, chat integrations (`@` file palette, clickable paths) |

## Tech stack

**Backend**: Bun + Hono + SQLite (bun:sqlite) + Drizzle ORM + Better Auth + croner. AI provider primitives are native, organized by capability in `src/server/llm/{llm,embedding,image,search,stt,tts,core}/`; plugins consume `@hivekeep/sdk`. (Vercel AI SDK was removed pre-2.0.)
**Frontend**: React + Vite + Tailwind CSS + shadcn/ui + i18next
**Single process, single DB file, single Docker container. Zero external infrastructure.**

## Key conventions

### Naming

- Files: `kebab-case.ts` / Components: `PascalCase.tsx`
- Types/Interfaces: `PascalCase` / Functions: `camelCase` / Constants: `SCREAMING_SNAKE_CASE`
- DB tables: `snake_case` / API routes: `kebab-case` / Env vars: `SCREAMING_SNAKE_CASE`

### Imports

Use absolute paths with tsconfig aliases:
```typescript
import { buildSystemPrompt } from '@/server/services/prompt-builder'
import type { Agent } from '@/shared/types'
```
No index barrels in deep folders — use explicit imports.

### Shared types

Any type used by both client and server goes in `src/shared/types.ts`. Any constant shared between client and server goes in `src/shared/constants.ts`.

### API errors

All API routes return JSON. Errors follow this format:
```json
{ "error": { "code": "ERROR_CODE", "message": "Human-readable description" } }
```

### i18n

- Base language: English (`en.json`). Supported UI locales: `en`, `fr`, `es`, `de`, `pt-BR`, `zh-CN`, `ja`, `ru`, `it`, `pl` (key parity enforced by `bun scripts/check-locales.ts`; no em-dashes in locale strings)
- UI language (`user_profiles.language`) is decoupled from the Agent communication language (`user_profiles.agent_language`, any `AGENT_LANGUAGES` code, null = follow UI language)
- Key convention: `namespace.element.action` (e.g. `sidebar.agents.title`)
- Use `useTranslation()` hook — never hardcode text in JSX
- Language detected from `user_profiles.language`, not the browser

### Database

- All PKs are UUIDs (text)
- All timestamps are Unix integers (milliseconds)
- Booleans stored as integer (0/1)
- Complex objects stored as text (JSON serialized)
- Better Auth tables (`user`, `session`, `account`, `verification`) are managed by Better Auth — never modify them directly

### Authentication

- Better Auth with HTTP-only cookie sessions
- Middleware on all `/api/*` routes except `/api/auth/*` and `/api/onboarding/*`
- First user created during onboarding gets `admin` role

### Design system

**Before building any frontend page or component**, read and follow the existing design system (it is already built — follow it, don't reinvent it):

| Reference | What it provides |
|---|---|
| `src/client/pages/design-system/DesignSystemPage.tsx` | Live showcase of every component, variant, animation, and pattern — **this is the source of truth for how UI should look and behave** |
| `src/client/styles/globals.css` | All design tokens (colors, radii, spacing), palette overrides, utility classes (`glass-strong`, `gradient-primary`, `gradient-border`, `btn-shine`, etc.), and keyframe animations |
| `src/client/components/ui/` | shadcn/ui components — always use these instead of creating custom ones. Many have custom `variant` props (e.g. `Progress`, `Slider`, `Button`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarlBurroW/hivekeep](https://github.com/MarlBurroW/hivekeep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
