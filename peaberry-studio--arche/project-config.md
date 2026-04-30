---
trigger: always_on
description: Mandatory guide for coding agents (Claude Code, Cursor, OpenCode, etc.) working in this repository. Read it before making any change.
---

# AGENTS.md

Mandatory guide for coding agents (Claude Code, Cursor, OpenCode, etc.) working in this repository. Read it before making any change.

## What Arche Is

Arche is a platform of specialized AI agents with isolated per-user workspaces. Each workspace is an OpenCode container with access to a shared Knowledge Base (Obsidian vault) and a catalog of configurable agents.

**Main components:**

- `apps/web/` - Next.js 16 app (React 19, TypeScript). It is the BFF (Backend for Frontend), the UI, and the container spawner.
- `apps/web/kickstart/` - Kickstart catalogs and templates (agents, KB skeletons, generated `AGENTS.md`).
- `infra/` - Infrastructure: Podman Compose, Ansible deployer, workspace image.
- `scripts/` - Deployment scripts for KB and config bare repositories.

## Technical Stack

- **Framework:** Next.js 16 + React 19 + TypeScript 5 (strict mode)
- **Styling:** Tailwind CSS 4 + shadcn/ui (Radix primitives)
- **DB:** PostgreSQL 16 + Prisma 7
- **Auth:** HTTP-only sessions with Argon2 + TOTP 2FA
- **Encryption:** AES-256-GCM for connectors and instance passwords
- **Containers:** Podman + Traefik + docker-socket-proxy
- **Package manager:** pnpm (no npm, no yarn)
- **Tests:** Vitest 3
- **Lint:** ESLint 9

---

## General Rules

### 1. Do not invent, ask

- If you cannot find the information you need in the code or docs, ask the user.
- Do not invent function names, endpoints, environment variables, or paths you did not verify.
- Do not assume a pattern exists: read the code before reproducing it.

### 2. Read before writing

- Always read the file you are going to modify before editing it.
- Understand surrounding context: what the module does, who imports it, what pattern it follows.
- Review nearby files to keep consistency.

### 3. Minimum necessary change

- Do only what was requested. Do not refactor or "improve" adjacent code, and do not add docstrings/comments unless requested.
- Do not add error handling, validation, or fallbacks for scenarios that cannot happen.
- Do not create premature abstractions or helpers for one-off operations.
- If you remove something unused, remove it completely (no `// removed`, no `_unused` vars, no compatibility re-exports).

### 4. Do not break working code

- Run tests before and after your changes: `pnpm test` from `apps/web/`.
- If tests fail because of your change, fix them before finishing.
- Do not disable tests or git hooks (`--no-verify`) unless the user explicitly asks.

---

## Code Conventions

### Naming

| Element | Convention | Example |
|----------|-----------|---------|
| Files | kebab-case | `agent-card.tsx`, `workspace-shell.tsx` |
| React components | PascalCase | `AgentCard`, `WorkspaceShell` |
| Functions / variables | camelCase | `startInstance`, `isConnected` |
| Types / interfaces | PascalCase | `AgentCardProps`, `SpawnerActionResult` |
| Constants | SCREAMING_SNAKE_CASE | `MIN_LEFT_PX`, `IDLE_TIMEOUT_MS` |
| Hooks | camelCase with `use` prefix | `useWorkspace`, `useWorkspaceTheme` |

### Imports

Required order:

```tsx
// 1. React / Next.js libraries
import { useCallback, useState } from 'react'
import { NextRequest, NextResponse } from 'next/server'

// 2. External dependencies
import { PrismaClient } from '@prisma/client'

// 3. Internal imports using @/
import { Button } from '@/components/ui/button'
import { cn } from '@/lib/utils'
import { useWorkspace } from '@/hooks/use-workspace'
```

- Always use the `@/` alias for internal imports. Never use relative paths like `../../lib/utils`.
- Sort alphabetically inside each group.

### React Components

```tsx
// Props type above the component
type AgentCardProps = {
  displayName: string
  agentId: string
  description?: string
}

export function AgentCard({ displayName, agentId, description }: AgentCardProps) {
  return (/* JSX */)
}
```

- Use `type` (not `interface`) for props, with `Props` suffix.
- Export named functions (`export function`), not `export default`.
- Mark client components with `'use client'` on the first line of the file.
- Mark server actions with `'use server'` on the first line.

### Styling

- Use Tailwind classes. Do not add custom CSS unless there is a real need.
- Combine classes with `cn()` from `@/lib/utils`:
  ```tsx
  <div className={cn('flex items-center', isActive && 'bg-primary')} />
  ```
- Base UI components (shadcn/ui) live in `src/components/ui/`. Do not modify them unless necessary; create wrappers when needed.

### TypeScript

- **Strict mode is enabled.** Do not use `any` or `as` without justification.
- Use explicit types at module boundaries (exports, API routes, server actions).
- Inside implementations, let TypeScript infer where possible.
- Use discriminated unions for error-capable results:
  ```tsx
  type Result =
    | { ok: true; data: Instance }
    | { ok: false; error: string }
  ```

### Error Handling

- Return typed `Result` objects instead of throwing exceptions.
- Validate only at system boundaries (user input, external APIs). Trust internal code.
- Do not swallow errors: if you catch, add context and rethrow/return.

---

## Architecture Patterns

### API Routes (BFF)

```
/api/u/[slug]/...      → User APIs (agents, connectors)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peaberry-studio/arche](https://github.com/peaberry-studio/arche) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
