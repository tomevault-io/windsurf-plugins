---
trigger: always_on
description: This repo is the community module registry for [Open Mercato](https://github.com/open-mercato/open-mercato). Each package under `packages/` is a publishable `@open-mercato/*` npm workspace. Every module is an **external extension** — it uses UMES extension points and MUST NOT modify core packages.
---

# Agents Guidelines — Official Modules

This repo is the community module registry for [Open Mercato](https://github.com/open-mercato/open-mercato). Each package under `packages/` is a publishable `@open-mercato/*` npm workspace. Every module is an **external extension** — it uses UMES extension points and MUST NOT modify core packages.

## Before Writing Code

1. Check the Task Router below and read **all** matching guides.
2. Check `.ai/specs/` for an existing spec before starting.
3. Enter plan mode for non-trivial tasks (3+ steps or architectural decisions).
4. If no scaffold exists yet, run the `scaffold-module` skill before `implement-spec`.

## Task Router

| Task | Guide |
|------|-------|
| Scaffold a new module package from scratch | `.ai/skills/scaffold-module/SKILL.md` |
| Write or review a spec for a new module | `.ai/skills/spec-writing/SKILL.md` |
| Implement a spec into a scaffolded package | `.ai/skills/implement-spec/SKILL.md` |
| Test a module in the sandbox | `apps/sandbox` — use `yarn *` commands from root |
| Publish a preview build to local Verdaccio | `yarn registry:up && yarn publish:preview` |

## Typical Agentic Workflow

```
spec-writing  →  scaffold-module  →  implement-spec
```

1. **`spec-writing`** — design the module in `.ai/specs/`; commit to module name, feature set, and API surface before writing code.
2. **`scaffold-module`** — create the buildable package skeleton under `packages/<name>/`.
3. **`implement-spec`** — fill in entities, API routes, UI pages, events, and tests on top of the skeleton.

---

## Framework Reference

Community modules are built on top of the Open Mercato framework. This section tells the agent **what to write** when implementing a module.

### Package Layout

```
packages/<package-name>/
├── package.json              # @open-mercato/<package-name>, publishable
├── tsconfig.json             # extends ../../tsconfig.base.json
├── build.mjs                 # esbuild build script (copy from test-package)
├── watch.mjs                 # watch mode wrapper
├── jest.config.cjs           # jest + ts-jest config
└── src/
    ├── index.ts              # barrel: export { metadata } from './modules/<moduleId>/index'
    └── modules/
        └── <moduleId>/       # snake_case — this is the module ID
            ├── index.ts      # ModuleInfo metadata + re-exports
            ├── acl.ts        # Feature definitions
            ├── setup.ts      # Tenant initialization
            ├── di.ts         # DI registrar (optional)
            ├── events.ts     # Typed event declarations (optional)
            ├── notifications.ts          # Notification types (optional)
            ├── notifications.client.ts   # Client notification renderers (optional)
            ├── translations.ts           # Translatable field declarations (optional)
            ├── search.ts                 # Search indexing config (optional)
            ├── ai-tools.ts               # MCP AI tool definitions (optional)
            ├── data/
            │   ├── entities.ts           # MikroORM entities
            │   ├── validators.ts         # Zod validation schemas
            │   ├── extensions.ts         # Entity extensions / cross-module links
            │   └── enrichers.ts          # Response enrichers
            ├── api/
            │   ├── interceptors.ts       # API route interception hooks
            │   └── <method>/
            │       └── <path>.ts         # API route handler
            ├── backend/
            │   └── <path>/
            │       ├── page.tsx          # React page component ('use client')
            │       └── page.meta.ts      # Page metadata (auth, features, title)
            ├── subscribers/
            │   └── <name>.ts             # Event subscriber
            ├── workers/
            │   └── <name>.ts             # Background worker
            └── widgets/
                ├── injection/
                │   └── <widget-name>/
                │       └── widget.ts     # Injection widget definition
                ├── injection-table.ts    # Widget-to-slot mappings
                └── components.ts         # Component replacement definitions
```

### Auto-Discovery Paths

The framework auto-discovers files by path convention — no manual registration needed:

| Path | Discovered as | URL |
|------|--------------|-----|
| `backend/<path>.tsx` | Backend admin page | `/backend/<path>` |
| `backend/page.tsx` | Module index page | `/backend/<moduleId>` |
| `api/<METHOD>/<path>.ts` | API route | `/api/<path>` (matched by HTTP method) |
| `subscribers/*.ts` | Event subscriber | Auto-wired on module load |
| `workers/*.ts` | Background worker | Auto-wired on module load |

### Module Convention Files

#### `src/modules/<moduleId>/index.ts`
```ts
import type { ModuleInfo } from '@open-mercato/shared/modules/registry'

export const metadata: ModuleInfo = {
  name: '<moduleId>',
  title: '<Human Title>',
  description: '<One sentence>',
  ejectable: true,   // optional — allows consumers to eject source
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-mercato/official-modules](https://github.com/open-mercato/official-modules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
