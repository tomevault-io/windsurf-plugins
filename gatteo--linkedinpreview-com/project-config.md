---
trigger: always_on
description: - Components: kebab-case files (e.g., `editor-panel.tsx`, `auth-provider.tsx`)
---

# Conventions

## Naming

### Files & Folders

- Components: kebab-case files (e.g., `editor-panel.tsx`, `auth-provider.tsx`)
- Utilities: kebab-case (e.g., `draft-url.ts`, `content-scoring.ts`)
- Routes/pages: kebab-case folders (e.g., `app/dashboard/editor/page.tsx`)
- Hooks: kebab-case with `use-` prefix (e.g., `use-drafts.ts`, `use-branding.ts`)
- Config: kebab-case (e.g., `config/site.ts`, `config/ai.ts`)
- Types: kebab-case (e.g., `types/blog.ts`)
- API route co-located files: `route.schema.ts` (Zod schemas), `route.utils.ts` (utility functions)
- No index.ts barrel files - import directly from the source file

### Variables & Functions

- Variables and functions: `camelCase` (e.g., `draftCount`, `parseBranding`)
- Module-level constants: `UPPER_SNAKE_CASE` (e.g., `MAX_FILE_SIZE`, `SYSTEM_PROMPT`, `POST_FORMATS`)
- Event handlers: prefix with `handle` (e.g., `handleCopy`, `handleFileChange`, `handleSubmit`)
- Boolean variables: prefix with `is`, `has`, `should`, `can` (e.g., `isLoading`, `hasError`, `canEdit`)
- Component props types: suffix with `Props` (e.g., `EditorPanelProps`)

## Code Style

- **Imports**: Enforced by `@ianvs/prettier-plugin-sort-imports`. Order: react, next, third-party, then internal groups (`@/env`, `@/types`, `@/config`, `@/lib`, `@/hooks`, `@/components/ui`, `@/components`, `@/styles`, `@/app`), then relative. Blank line between external and internal groups.
- **Exports**: Named exports only. Default exports only for `page.tsx`, `layout.tsx`, and route handlers. Export types separately with `export type`.
- **Functions**: Arrow functions (`const fn = () => {}`) for utilities and handlers. Function declarations (`function Component()`) for React components.
- **Types**: Use `type` keyword for all type definitions (not `interface`). Inline simple prop types directly in function signature if 3 or fewer props.
- **Comments**: Only when logic is non-obvious. No JSDoc on internal functions. No chatty or referential comments. Structural section labels (e.g., `// -- State --`) are fine.
- **Formatting**: Prettier config - 4-space indent, no semicolons, single quotes, jsxSingleQuote, 120 char width, bracketSameLine. Class sorting by `prettier-plugin-tailwindcss`.

## Component Patterns

Structure within each component file:

1. Imports
2. Type definitions
3. Component function
4. Helper functions below the component

- **Props**: Destructure in function signature. Required props first, optional last. Use `type Props = { ... }`.
- **State**: Local state with `useState`. No global state library. Shared state via React Context (AuthProvider pattern). URL state via `useSearchParams`.
- **Styling**: Tailwind classes directly on elements. Conditional classes via `cn()` from `lib/utils.ts`. Never CSS-in-JS. Never inline styles. Never CSS modules.
- **File structure**: One component per file. Co-locate sub-components in the same directory. Group by feature (`dashboard/`, `tool/`, `feedback/`).
- **No inline utilities**: Utility/helper functions (date formatting, color mapping, text extraction, data migration) belong in `lib/`, not in component files. Component files should only contain React components and component-specific display constants.
- **Server vs Client**: Server components by default. Add `'use client'` only for interactivity (event handlers, hooks, browser APIs). Keep `'use client'` boundaries as small as possible.
- **Static generation**: Use static site generation (SSG) everywhere possible. Pages without dynamic per-request data should be statically generated at build time. Use `generateStaticParams()` for dynamic routes (blog posts, compare pages). Only use server-side rendering when the page depends on request-time data (e.g. cookies, search params).
- **Shared components**: Reuse existing components before creating new ones. Extract shared UI patterns into `components/ui/` or feature-level shared components. Avoid duplicating similar layouts, cards, or interactive patterns across pages.
- **Dynamic imports**: Use `dynamic(() => import(...), { ssr: false })` for client-heavy components (TipTap editor). Never import SSR-false components in server components directly.

## Responsive Layout

The landing page must not scroll horizontally at 320px. A single element wider than the
viewport shifts the whole page and makes every section look broken, so the failure is never
local to the component that caused it.

- **Fixed-width decorative mockups** (browser chrome, sidebars, streak grids) set the page's
  minimum width unless every hardcoded `w-[...]`/`size-[...]` has a mobile branch. Give the
  mockup a smaller mobile variant (`w-[118px] sm:w-[150px]`), `truncate` any long unbroken
  string, and `min-w-0` the grid/flex item that holds it - grid and flex items default to
  `min-width: auto` and refuse to shrink below their content.
- **`flex-1` does not wrap.** It resolves to `flex: 1 1 0%`, so the item's hypothetical size
  is 0, it always "fits" the current line, and `flex-wrap` never fires - the item is squeezed
  below its content width instead. A flex child that should drop to its own row needs a real
  `min-w-[...]` floor per breakpoint.
- **Unbreakable phrases set the type floor.** When a headline glues words together

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gatteo/linkedinpreview.com](https://github.com/gatteo/linkedinpreview.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
