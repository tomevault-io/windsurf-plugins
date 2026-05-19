---
trigger: always_on
description: You are a professional software engineer. All code must follow best practices: accurate, readable, clean, and efficient.
---

# Sim Development Guidelines

You are a professional software engineer. All code must follow best practices: accurate, readable, clean, and efficient.

## Global Standards

- **Linting / Audit**: `bun run check:api-validation` must pass on PRs. Do not introduce route-local boundary Zod schemas, direct route Zod imports, or ad-hoc client wire types — see "API Contracts" and "API Route Pattern" below
- **Logging**: Import `createLogger` from `@sim/logger`. Use `logger.info`, `logger.warn`, `logger.error` instead of `console.log`
- **Comments**: Use TSDoc for documentation. No `====` separators. No non-TSDoc comments
- **Styling**: Never update global styles. Keep all styling local to components
- **ID Generation**: Never use `crypto.randomUUID()`, `nanoid`, or `uuid` package. Use `generateId()` (UUID v4) or `generateShortId()` (compact) from `@sim/utils/id`
- **Package Manager**: Use `bun` and `bunx`, not `npm` and `npx`

## Architecture

### Core Principles

1. Single Responsibility: Each component, hook, store has one clear purpose
2. Composition Over Complexity: Break down complex logic into smaller pieces
3. Type Safety First: TypeScript interfaces for all props, state, return types
4. Predictable State: Zustand for global state, useState for UI-only concerns

### Root Structure

```
apps/
├── sim/                    # Next.js app (UI + API routes + workflow editor)
│   ├── app/                # Next.js app router (pages, API routes)
│   ├── blocks/             # Block definitions and registry
│   ├── components/         # Shared UI (emcn/, ui/)
│   ├── executor/           # Workflow execution engine
│   ├── hooks/              # Shared hooks (queries/, selectors/)
│   ├── lib/                # App-wide utilities
│   ├── providers/          # LLM provider integrations
│   ├── stores/             # Zustand stores
│   ├── tools/              # Tool definitions
│   └── triggers/           # Trigger definitions
└── realtime/               # Bun Socket.IO server (collaborative canvas)
    └── src/                # auth, config, database, handlers, middleware,
                            # rooms, routes, internal/webhook-cleanup.ts

packages/
├── audit/                  # @sim/audit — recordAudit + AuditAction + AuditResourceType
├── auth/                   # @sim/auth — @sim/auth/verify (shared Better Auth verifier)
├── db/                     # @sim/db — drizzle schema + client
├── logger/                 # @sim/logger
├── realtime-protocol/      # @sim/realtime-protocol — socket operation constants + zod schemas
├── security/               # @sim/security — safeCompare
├── tsconfig/               # shared tsconfig presets
├── utils/                  # @sim/utils
├── workflow-authz/         # @sim/workflow-authz — authorizeWorkflowByWorkspacePermission
├── workflow-persistence/   # @sim/workflow-persistence — raw load/save + subflow helpers
└── workflow-types/         # @sim/workflow-types — pure BlockState/Loop/Parallel/... types
```

### Package boundaries

- `apps/* → packages/*` only. Packages never import from `apps/*`.
- Each package has explicit subpath `exports` maps; no barrels that accidentally pull in heavy halves.
- `apps/realtime` intentionally avoids Next.js, React, the block/tool registry, provider SDKs, and the executor. CI enforces this via `scripts/check-monorepo-boundaries.ts` and `scripts/check-realtime-prune-graph.ts`.
- Auth is shared across services via the Better Auth "Shared Database Session" pattern: both apps read the same `BETTER_AUTH_SECRET` and point at the same DB via `@sim/db`.

### Naming Conventions

- Components: PascalCase (`WorkflowList`)
- Hooks: `use` prefix (`useWorkflowOperations`)
- Files: kebab-case (`workflow-list.tsx`)
- Stores: `stores/feature/store.ts`
- Constants: SCREAMING_SNAKE_CASE
- Interfaces: PascalCase with suffix (`WorkflowListProps`)

## Imports

**Always use absolute imports.** Never use relative imports.

```typescript
// ✓ Good
import { useWorkflowStore } from '@/stores/workflows/store'

// ✗ Bad
import { useWorkflowStore } from '../../../stores/workflows/store'
```

Use barrel exports (`index.ts`) when a folder has 3+ exports. Do not re-export from non-barrel files; import directly from the source.

### Import Order

1. React/core libraries
2. External libraries
3. UI components (`@/components/emcn`, `@/components/ui`)
4. Utilities (`@/lib/...`)
5. Stores (`@/stores/...`)
6. Feature imports
7. CSS imports

Use `import type { X }` for type-only imports.

## TypeScript

1. No `any` - Use proper types or `unknown` with type guards
2. Always define props interface for components
3. `as const` for constant objects/arrays
4. Explicit ref types: `useRef<HTMLDivElement>(null)`

## Components

```typescript
'use client' // Only if using hooks

const CONFIG = { SPACING: 8 } as const

interface ComponentProps {
  requiredProp: string
  optionalProp?: boolean
}

export function Component({ requiredProp, optionalProp = false }: ComponentProps) {
  // Order: refs → external hooks → store hooks → custom hooks → state → useMemo → useCallback → useEffect → return
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simstudioai/sim](https://github.com/simstudioai/sim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
