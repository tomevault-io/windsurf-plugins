---
trigger: always_on
description: Next.js 14+ (App Router) web UI for [Cadence Workflow](https://cadenceworkflow.io/). TypeScript throughout. Talks to a Cadence backend over gRPC.
---

# cadence-web — Claude Context

## Project Overview

Next.js 14+ (App Router) web UI for [Cadence Workflow](https://cadenceworkflow.io/). TypeScript throughout. Talks to a Cadence backend over gRPC.

## Key Commands

```bash
npm run dev          # Dev server on :8088
npm run test             # All tests + type tests (85% coverage threshold enforced)
npm run test:unit:browser  # Browser (jsdom) tests only
npm run test:unit:node     # Node (server-side) tests only
npm run lint         # ESLint
npm run typecheck    # tsc --noemit
npm run build        # Production build
```

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14 App Router |
| Language | TypeScript |
| UI components | [Base Web (baseui)](https://baseweb.design/) — Uber's design system |
| Styling | `baseui`'s `styled()` utility (Styletron under the hood) |
| Data fetching | TanStack Query (React Query v5) with Next.js experimental RSC support |
| Backend protocol | gRPC via `@grpc/grpc-js` |
| Form validation | React Hook Form + Zod |
| Testing | Jest — two separate configs: `browser` (jsdom) and `node` |
| Test utilities | `@testing-library/react`, MSW (Mock Service Worker) for API mocking |
| Type testing | `tstyche` |

## Directory Structure

```
src/
  app/              # Next.js App Router — pages, layouts, error/loading boundaries
    (Home)/         # Route group for main UI pages
    api/            # API route handlers (server-side, call Cadence gRPC)
  components/       # Abstract, reusable UI components (no business logic)
  views/            # Feature modules (complex, page-level components)
    shared/         # Cadence-specific reusable components (has business logic)
  hooks/            # Shared custom React hooks
  providers/        # React context providers
  route-handlers/   # Server-side route handlers
  utils/            # Pure utility functions
  config/           # App configuration
  test-utils/       # Shared test helpers (excluded from coverage)
  __generated__/    # Generated protobuf types — never edit manually
```

## Component vs View/Shared Distinction

**`src/components/`** — Generic UI, no Cadence knowledge:
- Buttons, tables, inputs, layout primitives
- Could be used in any app

**`src/views/shared/`** — Cadence business logic components:
- Components that fetch/render Cadence objects (workflows, domains, etc.)
- Understand Cadence data structures and APIs
- Examples: `WorkflowStatusTag`, `DomainStatusTag`

## File Conventions

Every component/view follows a consistent multi-file pattern:

```
component-name/
  component-name.tsx           # Component implementation
  component-name.styles.ts     # Styletron/baseui styles
  component-name.types.ts      # TypeScript types/interfaces
  __tests__/
    component-name.test.tsx    # Tests
  __fixtures__/                # Test data (excluded from coverage)
```

### Naming
- Files and directories: **kebab-case**
- Components/types: **PascalCase**
- Constants: **UPPER_SNAKE_CASE**
- Functions/variables: **camelCase**
- Path alias: `@/` resolves to `src/`

### Directory Nesting Limits
Max **2 levels** of component directories under `src/views/` and `src/components/`. `__tests__/` at level 3 is fine. Never go deeper.

```
src/views/
  domain-page/               # Level 1 ✓
    domain-page-header/      # Level 2 ✓
      __tests__/             # Level 3 ✓ (tests/fixtures/helpers only)
```

## Config and constants

- **Config** — Values and wiring that **forks are expected to change** between deployments or product variants (clusters, auth, feature visibility, columns).
- **Constants** — **Stable, reusable literals** that are **not** treated as fork overrides (shared labels, fixed format tokens, internal limits used the same way upstream and in forks). Prefer `*.constants.ts`, feature-local modules for cross-cutting fixed values.

## Styling Pattern

Use `baseui`'s `styled` utility — **not** raw Styletron or CSS modules:

```ts
// component.styles.ts
import { styled as createStyled, type Theme } from 'baseui';
import { type StyleObject } from 'styletron-react';

export const styled = {
  Container: createStyled('div', ({ $theme }: { $theme: Theme }): StyleObject => ({
    padding: $theme.sizing.scale600,
    color: $theme.colors.contentPrimary,
  })),
};
```

Always use theme tokens: `$theme.sizing.scale*` for spacing, `$theme.colors.*` for colors, `$theme.typography.*` for text.

For responsive styles, use `$theme.mediaQuery.*`:
```ts
{ [$theme.mediaQuery.medium]: { flexDirection: 'row' } }
```

Use `useStyletronClasses` hook for CSS object styles:
```ts
import useStyletronClasses from '@/hooks/use-styletron-classes';
const { cls } = useStyletronClasses(cssStyles);
```

Use `mergeOverrides` from `baseui/helpers/overrides` when extending baseui component overrides.

## TypeScript Conventions

- Prefer `type` over `interface`;
- Use generics with meaningful constraints for reusable components/hooks
- Never use `any` — use `unknown` and narrow
- Always explicitly type hook return values

```ts
export type Props = {
  required: string;
  optional?: number;
};
```

## URL Params and Encoding


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cadence-workflow/cadence-web](https://github.com/cadence-workflow/cadence-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
