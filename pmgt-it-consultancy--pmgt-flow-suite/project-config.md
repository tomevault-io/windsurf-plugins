---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A fullstack POS (Point of Sale) system for restaurant operations, built as a monorepo with web (Next.js 16) and mobile (React Native/Expo) frontends sharing a Convex backend. Features order management, product catalog with modifiers, table management, takeout workflows, discount/void processing, receipt printing, audit logging, and sales reporting.

## Commands

```bash
# Install dependencies (uses pnpm)
pnpm install

# Run all apps in development (web, native, backend)
pnpm dev

# Type checking across all packages
pnpm typecheck

# Lint and format (uses Biome, not ESLint/Prettier)
pnpm lint
pnpm format
pnpm check              # lint + format combined

# Build all packages
pnpm build

# Run backend tests (Vitest + convex-test)
cd packages/backend && pnpm vitest
cd packages/backend && pnpm vitest run    # single run, no watch

# Per-app commands
cd apps/web && pnpm lint
cd apps/native && pnpm ios
cd apps/native && pnpm android
cd apps/native && pnpm start
```

## Development Tooling

### Required Skills

These skills MUST be invoked via the `Skill` tool before writing or reviewing code in the matching domain. Do not rely on general knowledge — load the skill first so its current guidance is applied.

| When working on... | Invoke |
|--------------------|--------|
| `apps/web` (Next.js/React) | `vercel-react-best-practices`, `vercel-composition-patterns` |
| `apps/native` (React Native/Expo) | `vercel-react-native-skills`, `vercel-react-best-practices`, `vercel-composition-patterns` |
| Any new UI, visual design, layout, or styling (web or native) | `frontend-design` |
| New features, components, or behavior changes | `brainstorming` before implementation |
| Multi-step tasks with a spec | `writing-plans`, then `executing-plans` |
| Bugs, test failures, unexpected behavior | `systematic-debugging` |

Compose skills when multiple apply (e.g. a new native screen → `brainstorming` → `frontend-design` → `vercel-react-native-skills` + `vercel-composition-patterns` during implementation).

### Serena (Semantic Code Navigation)

Serena provides LSP-backed semantic tools. Prefer them over raw `Read` + `Grep` whenever the target is a symbol rather than a text blob — they're cheaper on context and more accurate across renames.

**Discovery flow (use this order):**
1. `get_symbols_overview` — get a file's top-level symbols before reading anything
2. `find_symbol` with `name_path` (e.g. `processPayment`, `OrderCard/render`) — jump directly to a definition; set `include_body: true` only when you need the implementation
3. `find_referencing_symbols` — find all call sites / usages before renaming or changing a signature
4. `search_for_pattern` — fallback for non-symbol text (strings, comments, config keys)
5. `Read` on a full file only when symbolic exploration isn't enough (e.g. config files, JSON, markdown)

**Editing:**
- `replace_symbol_body` — rewrite a full function/class body by name path (safer than `Edit` for large symbols)
- `insert_after_symbol` / `insert_before_symbol` — add new symbols adjacent to existing ones
- `rename_symbol` — rename across the project (propagates to all references)
- `safe_delete_symbol` — delete a symbol only when no references remain

**Project memory:**
- `list_memories` / `read_memory` / `write_memory` — Serena's own persistent memory (separate from auto-memory). Use for durable project knowledge that should survive conversations: architectural decisions, domain glossaries, non-obvious invariants. Not for conversation state or todos.
- Use the `save-serena` skill when finishing a session to capture progress.

**Scoping:** Always pass `relative_path` (file or directory) to symbolic searches when you know the area — avoids whole-repo scans.

**Anti-patterns:**
- Don't `Read` an entire file and then call symbolic tools on it — you already have the content
- Don't `search_for_pattern` for a known symbol name — use `find_symbol`
- Don't `Edit` a whole function body — use `replace_symbol_body`
- Don't use text-based rename — use `rename_symbol`

## Architecture

### Monorepo Structure
- **apps/web** — Next.js 16 App Router, Tailwind CSS v4, Radix UI components, React Hook Form + Zod
- **apps/native** — React Native 0.81 + Expo 54, Tamagui (UI/styling), React Navigation (bottom tabs + stack), Zustand for local state, Bluetooth ESC/POS receipt printing
- **packages/backend** — Convex backend (schema, queries, mutations, actions, tests)
- **packages/shared** — Shared utilities

Managed with **Turborepo** (`turbo.json`) and **pnpm** workspaces.

### Data Flow
Both frontends import from `@packages/backend` and use Convex client hooks (`useQuery`, `useMutation`) for real-time data. Authentication uses `@convex-dev/auth` with Convex Auth tables.

Money handling follows the current app behavior, not the older centavo-only assumption:
- Product prices, modifier adjustments, discounts, and tendered cash are treated as peso amounts with decimals.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pmgt-it-consultancy/pmgt-flow-suite](https://github.com/pmgt-it-consultancy/pmgt-flow-suite) — distributed by [TomeVault](https://tomevault.io/claim/pmgt-it-consultancy).
<!-- tomevault:4.0:windsurf_rules:2026-04-17 -->
