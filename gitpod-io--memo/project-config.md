---
trigger: always_on
description: This repository is entirely agent-generated. No human writes code here.
---

# AGENTS.md

This repository is entirely agent-generated. No human writes code here.

## Stack

- Next.js 16 (App Router), TypeScript (strict), Tailwind CSS, shadcn/ui
- Supabase: database (PostgreSQL), auth, realtime — via `@supabase/supabase-js`
- Sentry (`@sentry/nextjs`) for error tracking
- Storybook 8 (`@storybook/react-vite`) for component development and visual documentation
- Vitest for unit/integration tests, Playwright for E2E and visual regression
- Deployed on Vercel, domain: software-factory.dev

## Project Structure

```
src/app/           → Pages and API routes (App Router)
src/components/    → Reusable UI components (one per file, named exports)
src/components/ui/ → shadcn/ui components (do not edit)
src/lib/           → Utilities, types, constants
src/lib/supabase/  → Supabase clients (client.ts, server.ts, proxy.ts)
supabase/migrations/ → Database migrations
.storybook/        → Storybook config (main.ts, preview.ts, preview-head.html)
.agents/           → Agent knowledge base (architecture, conventions, design)
.ona/              → Automation definitions and skills
docs/              → Product spec, decisions
metrics/           → Daily/weekly metrics snapshots
```

## Rules

- Server components by default. `"use client"` only for hooks, event handlers, or browser APIs.
- No `any`. No `@ts-ignore`. No `as` casts unless unavoidable (comment why).
- No ORMs — use `@supabase/supabase-js` for all database operations.
- No custom CSS — Tailwind utility classes only.
- Check shadcn/ui before building custom components.
- Named exports only. No default exports.
- Conventional commits: `feat|fix|chore|docs|test|refactor(scope): description`
- PRs with type `feat` or `fix` must reference an issue: `Closes #N`. Chore PRs (metrics, docs, deps) do not require an issue.
- **Issue-first workflow:** Before creating a `feat` or `fix` PR, create a GitHub issue first (or find an existing one). Label it `status:in-progress` immediately. Add `Closes #N` to the PR description. This prevents the PR Reviewer from blocking the merge.
- **Exception — `ona-user` PRs:** PRs created via interactive Ona sessions (user prompts) may use the `ona-user` label instead of linking an issue. The PR Reviewer will merge these without requiring `Closes #N`.
- UI component PRs must include co-located `*.stories.tsx` files covering default state, variants, and interactive states. The PR Reviewer will request stories if they are missing.
- Database changes require a migration: `npx supabase migration new <name>`
- Environment variables: `NEXT_PUBLIC_` prefix only for browser-safe values.

## Testing

### Testing pyramid

Each test type has a specific purpose. Do not substitute one for another.

- **Unit tests (Vitest):** utility functions, non-trivial logic, API route handlers, data transformations. Mock external dependencies (Supabase, fetch).
- **Component tests (Vitest + jsdom):** render components, verify callbacks fire, check state transitions. Use for logic-heavy components that don't need a real browser.
- **E2E tests (Playwright):** interactive features, critical user flows, new pages. **Mandatory** for any PR that adds or modifies interactive UI — see below.
- **Visual regression (Playwright):** `pnpm test:visual` — screenshots Storybook stories and compares against committed baselines in `e2e/visual-regression.spec.ts-snapshots/`
- **Static analysis tests (Vitest):** structural convention enforcement only (e.g., "no bare catch blocks", "no `@ts-ignore`", migration validation). **Never** for verifying feature behavior — see below.
- Skip tests for trivial layout-only components.

### E2E tests are mandatory for interactive UI

Any PR that adds or modifies components with user interactions (`onClick`, `onChange`, `onBlur`, `onKeyDown`, dropdowns, dialogs, popovers, drag-and-drop) **must** include E2E tests in the same PR. Do not defer E2E tests to follow-up issues.

E2E tests are required for:
- Buttons, dropdowns, and menus that trigger actions
- Inline editing (click-to-edit cells, inputs that appear on interaction)
- Dialogs and popovers (open, interact, dismiss)
- Drag-and-drop (editor blocks, sidebar pages, column reorder)
- Multi-step flows (auth, page creation, workspace switching)
- Any feature where the bug would only manifest in a real browser

When unit tests are sufficient (no E2E needed):
- Pure functions and utilities
- API route handlers (mock Supabase)
- Data transformations (markdown conversion, tree building)
- Components with no event handlers (pure render)

### PRs that change interaction flows must update E2E tests

Any PR that changes a user-facing interaction flow (button behavior, dialog flow, dropdown behavior, keyboard shortcuts) must update **all** affected E2E tests in the same PR. Search for all references: `grep -r '<component-name>\|<selector>' e2e/` before committing.

Do not merge interaction changes without verifying E2E tests pass. The PR Reviewer will block PRs that change interactive components without corresponding E2E test updates.

### No source-grep tests for feature behavior


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gitpod-io/memo](https://github.com/gitpod-io/memo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
