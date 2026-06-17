---
trigger: always_on
description: You are a professional software engineer. All code must follow best practices: accurate, readable, clean, and efficient.
---

# Tripwire Development Guidelines

You are a professional software engineer. All code must follow best practices: accurate, readable, clean, and efficient.

## Global Standards

- **Comments**: Use JSDoc for exported functions/types only. No ASCII divider comments (`// ─── Section ───`). No decorative multi-line doc comments.
- **Styling**: Never use local classnames on componenets unless you really need to. Keep all styling to global tokens.
- **Package Manager**: Use `pnpm`, not `npm` or `yarn`.
- **Lint**:
  - **Repo root** (`pnpm lint`): runs **ESLint** on the monorepo (`eslint.config.js`), then **`pnpm --filter @tripwire/web lint`** (**Biome** rules for `apps/web` only — structural/import restrictions such as restricted elements).
  - **`apps/web` alone**: `pnpm --filter @tripwire/web lint` is Biome only; use root `pnpm lint` before CI-style checks.
  - ESLint ignores generated and tooling dirs: `node_modules`, `dist`, `.output`, `.turbo`, `.claude`, `.cursor`, `.tanstack`, `.nitro`, `.vinxi`, `**/routeTree.gen.ts`, etc.
- **Server/Client Boundary**: Never import `@tripwire/db` or `@tripwire/core` barrels in client code. Use subpath imports (`@tripwire/db/schema/rule-meta`, `@tripwire/core/rules/signal-registry`, `@tripwire/core/workflow-registry`) for pure-data modules. `import type` is always safe.
- **No Effects for Prop Sync**: Never use `useEffect` to sync props to state. Use `key` props to remount, derived state, or controlled components.

## Architecture

### Core Principles

1. Single Responsibility: Each component, hook, store has one clear purpose
2. Composition Over Complexity: Break down complex logic into smaller pieces
3. Type Safety First: TypeScript interfaces for all props, state, return types
4. No Helpers in Components: Put utility functions in `apps/web/src/lib/` or `packages/*/src/`, never inside component or tool files
5. Never use raw html components (<button>, <input>, <select><option></select>, etc). Always use their react counterparts defined in `apps/web/src/components/ui` (shared primitives such as `Button` live in `@tripwire/ui` and are re-exported under `#/components/ui` for the app).
   If you're unable to find the right component, explore the coss ui directory. `https://coss.com/ui/docs`
6. Never define a raw vector (`<svg`) outside `apps/web/src/components/icons/` (for the app) or `packages/ui/src/icons/` (for shared UI). Import icon components from there and follow the same patterns as existing icons.
7. Never introduce a raw inline type to a component-level or route-level file. Keep it scoped to a generally existing file which the majority of other similar types live.
8. Absolutely do not spam single line comments above every change. Comments should be sparce, contain no em dashes or overly verbose language patterns, and should ONLY be used to give important context about programmatic decisions, nuance, etc. Never to just explain what a line does, especially given that every function, const, etc should be highly readable as is.

### Root Structure

```
apps/
└── web/                       # TanStack Start app (UI + API routes + tRPC)
    ├── src/
    │   ├── components/        # Shared UI components
    │   │   ├── automations/   # Workflow editor, node types, templates
    │   │   ├── chat/          # AI chat panel, context, thread
    │   │   ├── layout/        # App shell, empty states
    │   │   ├── rules/         # Rule cards, custom rules tab, rule builder
    │   │   └── ui/            # Base UI primitives (button, dialog, etc.)
    │   ├── integrations/      # tRPC client/server setup + routers
    │   ├── lib/               # App-wide utilities
    │   ├── routes/            # TanStack file-based routes
    │   │   ├── _app/          # Authenticated app routes
    │   │   └── api/           # API routes (chat, webhook, tools, auth)
    │   └── types/             # Shared type declarations
    └── autumn.config.ts       # Billing/plan configuration

packages/
├── ai/                        # @tripwire/ai — system prompt, model config, credit metering
├── auth/                      # @tripwire/auth — Better Auth setup
├── core/                      # @tripwire/core — business logic (pipeline, scoring, rules, operations)
├── db/                        # @tripwire/db — Drizzle schema + client (Postgres)
├── env/                       # @tripwire/env — environment variable schemas
├── github/                    # @tripwire/github — GitHub API client
├── mcp/                       # @tripwire/mcp — MCP server adapter
├── ratelimit/                 # @tripwire/ratelimit — rate limiting
├── tools/                     # @tripwire/tools — tool definitions (MCP + chat)
└── ui/                        # @tripwire/ui — shared UI utilities (cn, etc.)
```

### Package Boundaries

- `apps/* -> packages/*` only. Packages never import from `apps/*`.
- Each package has explicit subpath `exports` in `package.json`. Use subpaths for client-safe imports to avoid pulling server-only dependencies (Drizzle, pg) into the browser bundle.
- Auth is shared via Better Auth with Autumn for billing.

### Naming Conventions

- Components: PascalCase (`RuleCardGrid`)
- Hooks: `use` prefix (`useWorkspace`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bountydotnew/tripwire](https://github.com/bountydotnew/tripwire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
