---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`furio-kit` is a React boilerplate by FurioLabs targeting corporate/enterprise frontends. It uses Next.js App Router with React Server Components as the primary rendering paradigm.

UI primitives (Atoms, Molecules) are **not owned by this codebase**. Each organization provides its own design system as an external package (`@org/ui-kit`). This application consumes that package through a thin adapter layer in `src/shared/ui/`.

## Technology Stack

- **Next.js 16+** — App Router, Turbopack dev server, Server Actions
- **React 19+** — React Compiler, Server Components by default
- **TypeScript** — strict mode
- **pnpm** — package manager
- **Biome 2+** — linting and formatting (replaces ESLint + Prettier)
- **Tailwind CSS v4** — utility-first styling; no config file, content scanning is automatic
- **`@tailwindcss/postcss`** — required PostCSS plugin for Tailwind v4
- **`clsx` + `tailwind-merge`** — combined via `cn()` in `shared/utils` for safe class merging
- **Zod** — runtime schema validation at system boundaries
- **TanStack Query** — client-side cache management and optimistic updates
- **Zustand** — client UI state (SSR-safe initialization required)
- **Vitest** — unit and integration tests
- **`@org/ui-kit`** — external design system package; provides Atoms and Molecules following Atomic Design

## Commands

```bash
pnpm dev          # dev server (Turbopack)
pnpm build        # production build
pnpm start        # production server
pnpm lint         # Biome lint
pnpm format       # Biome format
pnpm test         # Vitest run
pnpm test:watch   # Vitest watch mode
pnpm audit        # check for vulnerabilities
```

## Tailwind v4 Setup

Tailwind v4 differs from v3 in three ways relevant to this codebase:

- **No `tailwind.config.ts`** — content scanning is automatic; delete this file if it reappears
- **CSS import**: `app/globals.css` uses `@import "tailwindcss"` (not the v3 `@tailwind` directives)
- **PostCSS plugin**: `postcss.config.mjs` uses `'@tailwindcss/postcss'` (not `tailwindcss`)

## Design System: `@org/ui-kit`

Each organization deploying `furio-kit` supplies their own `@org/ui-kit` package. The package follows Atomic Design and exposes:

| Level | Examples | Notes |
|---|---|---|
| **Atoms** | `Button`, `Input`, `Badge`, `Icon`, `Label` | Stateless, fully styled primitives |
| **Molecules** | `FormField`, `Card`, `Modal`, `Tooltip` | Composed from Atoms |
| **Tokens** | CSS variables or a `tokens` export | Colors, spacing, typography |

`furio-kit` **never re-implements** components that exist in `@org/ui-kit`. It only wraps them.

### Adapter pattern (mandatory)

`@org/ui-kit` must never be imported directly from `features`, `entities`, `widgets`, or `views`. All consumption goes through adapter components in `shared/ui`.

```bash
shared/ui/
  Button/
    Button.tsx      ← "use client"; wraps @org/ui-kit Button
    index.ts
  Card/
    Card.tsx        ← Server Component; wraps @org/ui-kit Card
    index.ts
  index.ts          ← barrel: re-exports all adapters
```

Example adapter:

```tsx
// shared/ui/Button/Button.tsx
'use client'
import { Button as OrgButton } from '@org/ui-kit'
import type { ReactNode } from 'react'

export interface ButtonProps {
  children: ReactNode
  onClick?: () => void
  variant?: 'primary' | 'secondary' | 'ghost'
  disabled?: boolean
  type?: 'button' | 'submit' | 'reset'
}

export function Button({ children, variant = 'primary', ...rest }: ButtonProps) {
  return <OrgButton variant={variant} {...rest}>{children}</OrgButton>
}
```

Internal layers import from `shared/ui`, never from `@org/ui-kit` directly:

```ts
import { Button } from '@/shared/ui'   // correct
import { Button } from '@org/ui-kit'   // forbidden
```

### RSC compatibility of adapters

Assume `@org/ui-kit` components are Client Components unless the package explicitly states otherwise. Mark adapters `"use client"` when wrapping interactive primitives. Purely presentational adapters can remain Server Components.

## Rendering Architecture

### React Server Components (default)

All components are Server Components unless explicitly marked `"use client"`. Server Components:

- Fetch data directly (no `useEffect`, no TanStack Query)
- Access backend services, databases, and environment secrets
- Cannot use hooks, event handlers, or browser APIs
- Are never hydrated on the client — they produce static HTML

### Client Components (`"use client"`)

Mark a component `"use client"` only when it needs:

- React hooks (`useState`, `useEffect`, `useRef`, etc.)
- Browser APIs or event handlers
- Zustand stores or TanStack Query hooks

`"use client"` is a boundary, not a per-component flag — it propagates to all children. Push this boundary as deep as possible.

### Data Fetching

| Scenario | Approach |
|---|---|
| Initial page data | `async` Server Component — `await fetch()`/DB call directly |
| Mutations | Server Actions (`"use server"`) |
| Client-side cache / optimistic UI | TanStack Query (within Client Components) |
| SSR + client handoff | TanStack Query `HydrationBoundary` + `dehydrate()` in Server Components |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/furio-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
