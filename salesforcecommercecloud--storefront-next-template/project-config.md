---
trigger: always_on
description: Storefront template for Salesforce Commerce Cloud built with React Router v7, React, Tailwind CSS, and Vite. Integrates with Commerce Cloud via SCAPI.
---

# Template Retail App

Storefront template for Salesforce Commerce Cloud built with React Router v7, React, Tailwind CSS, and Vite. Integrates with Commerce Cloud via SCAPI.

This file is the single source of truth for AI coding agents (Claude Code, Cursor, Codex, etc.) working in this package. `CLAUDE.md` is a symlink to this file.

## Project Structure

- `./src/` — Application source code
  - `./src/routes/` — React Router file-based routes
  - `./src/components/` — React components (`./src/components/ui/` for Radix + Tailwind primitives)
  - `./src/lib/` — Shared utilities, hooks, business logic (adapters, adapter registry, decorators)
  - `./src/hooks/`, `./src/providers/` — React hooks and context providers
  - `./src/extensions/` — Optional feature extensions
  - `./src/locales/` — i18n translation files
  - `./src/types/config.ts` — Template-specific config types
- `./config.server.ts` — Configuration defaults
- `./.storybook/` — Storybook configuration
- `./public/` — Static assets
- `./docs/` — Detailed docs (see **Key Documentation** below)

## Common Commands

```bash
# Dev
pnpm dev                         # Dev server at http://localhost:5173
pnpm dev:debug                   # Dev server with Node debugger
pnpm storybook                   # Storybook at http://localhost:6006

# Build / deploy
pnpm build                       # Production build
pnpm preview                     # Preview production build
pnpm push                        # Deploy to Commerce Cloud Managed Runtime
pnpm generate:cartridge          # Extract Page Designer metadata

# Quality
pnpm typecheck
pnpm lint                        # Strict: --max-warnings 0 (CI enforces)
pnpm lint:fix
pnpm bundlesize:test             # Verify bundle size limits
pnpm lighthouse:ci

# Tests — prefer :agent variants for condensed output
pnpm test:agent                  # Unit tests (summary only)
pnpm test                        # Unit tests (verbose, with coverage)
pnpm test:watch
pnpm test src/components/foo     # Single file/dir

pnpm test-storybook:interaction:agent
pnpm test-storybook:a11y:agent
pnpm test-storybook:snapshot:agent

# UITargets
pnpm --filter template-retail-rsc-app dev:ui-targets        # Visual overlay showing targets
pnpm --filter template-retail-rsc-app smoke-test:generate   # Sync target-config.json (additive)
```

### Agent Command Summary

| Command | Purpose | Output |
|---------|---------|--------|
| `pnpm test:agent` | Unit tests | Last 30 lines |
| `pnpm test:agent:coverage` | Unit tests + coverage | Last 40 lines |
| `pnpm test-storybook:snapshot:agent` | Snapshot tests | Last 30 lines |
| `pnpm test-storybook:interaction:agent` | Interaction tests | Last 20 lines (PASS/FAIL only) |
| `pnpm test-storybook:a11y:agent` | A11y tests | Last 20 lines (violations only) |

## Performance & Data Rules

These rules take priority when designing routes, components, and state. Apply them as a checklist for every route module and every component that consumes async data. See [Data Fetching](./docs/README-DATA.md), [Loading States](./docs/README-SUSPENSE.md), [State Management](./docs/README-STATE.md), and [Performance](./docs/README-PERFORMANCE.md) for full context.

### Data Loading

1. **Server-load everything.** All initial data must come from server `loader` functions — never `useEffect`, `fetch`, or other client-side fetching for data needed on first render.
2. **Classify every data field per route.** Critical data (SEO, LCP, CLS, HTTP status) is `await`ed in the loader. Non-critical data is returned as an unresolved Promise. Interaction-driven data is fetched via `useFetcher` on user action.
3. **Never block the loader on non-critical data.** Return the Promise directly — don't `await` recommendations, reviews, or below-the-fold content.
4. **Export `shouldRevalidate` on routes with URL-driven filtering.** Prevent redundant loader re-execution when only search params change and the loader already handles them on the next navigation.
5. **No `clientLoader` or `clientAction`.** Only server `loader` and server `action` exports are permitted in route modules.

### Rendering & Visual Stability

6. **One `<Suspense>` boundary per async operation.** Never place multiple `use()` calls or `<Await>` components inside a single `<Suspense>` boundary — each deferred Promise gets its own boundary and its own skeleton. See [Suspense Boundary Granularity](./docs/README-SUSPENSE.md#suspense-boundary-granularity) for examples and anti-patterns.
7. **Skeleton screens for known layouts, spinners for indeterminate operations.** If the shape of the resolved content is known, use a skeleton. Spinners are only for global or unknown-layout loading states.
8. **Above the fold: avoid `fallback={null}` without reserving space.** Rendering nothing and then injecting content causes CLS. If no visual fallback is desired, the container must maintain explicit dimensions (`minHeight`, aspect ratio).
9. **Below the fold: prefer `fallback={null}` or a simple placeholder.** Users don't perceive layout shift for content they can't see, and complex skeletons add hydration cost without visible benefit.

### Mutations & Interactions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SalesforceCommerceCloud/storefront-next-template](https://github.com/SalesforceCommerceCloud/storefront-next-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
