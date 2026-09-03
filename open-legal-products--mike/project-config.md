---
trigger: always_on
description: These instructions apply to the entire repository. Keep changes focused,
---

# Repository Instructions

## Scope

These instructions apply to the entire repository. Keep changes focused,
preserve unrelated work in the tree, and follow the more detailed guidance in
`CONTRIBUTING.md` and `docs/` when working in a documented subsystem.

The repository requires Node.js 22 or newer and contains three applications:

- `frontend/`: Next.js web application.
- `backend/`: Express API, document processing, database access, and LLM
  integration.
- `word-addin/`: Microsoft Word task-pane add-in.

The root `e2e/` directory contains the web application's Playwright tests.

## UI and Design

Read `docs/design-system.md` before making a non-trivial UI change. It is the
source of truth for tokens, spacing, surfaces, primitives, and accessibility.

Informational labels and statuses should normally be plain text rather than
decorative pill badges unless a pill is specifically requested. This does not
apply to established interactive pill controls such as `PillButton`,
`TabPillButton`, and `OptionPill`.

Before creating UI markup, search these locations in order:

1. `frontend/src/app/components/ui/` contains reusable web primitives such as
   buttons, inputs, dropdowns, search, empty states, and liquid-glass surfaces.
2. `frontend/src/shared/ui/` contains framework-light components rendered by
   both the web app and Word add-in. Files use the `XxxUI.tsx` convention, with
   thin web adapters or re-exports in `frontend/src/app/components/ui/` where
   appropriate. Code here must not import from `frontend/src/app/`.
3. `frontend/src/app/components/shared/` contains web-app building blocks that
   understand the application shell, including `PageHeader`,
   `TablePrimitive`, `TableToolbar`, `FileDirectory`, and side panels.
4. `frontend/src/app/components/modals/` and
   `frontend/src/app/components/popups/` contain the established modal,
   confirmation, and warning patterns.
5. `frontend/src/app/components/<feature>/` contains feature-specific web
   components. Keep one-off feature behavior here instead of turning it into a
   primitive prematurely.

For add-in-only work, low-level controls live in `word-addin/src/shared/ui/`
and reusable task-pane compositions live in
`word-addin/src/taskpane/components/primitives/`. If a component must render in
both the web app and add-in, prefer `frontend/src/shared/ui/`. When adding a
cross-target file there, also add its `@source` entry to
`word-addin/src/taskpane/styles.css` so Tailwind discovers its classes.

The web app is configured for shadcn's `new-york` style in
`frontend/components.json`. If no existing primitive fits a non-trivial
interaction, add the shadcn component from the `frontend/` directory so it
lands in `src/app/components/ui/`. Do not add a new UI dependency when an
existing primitive or Tailwind can solve the problem.

Design changes should follow the existing liquid-glass theme, use borders
sparingly, and use Lucide icons. Prefer the tokens and shared class constants
in `frontend/src/app/globals.css` and
`frontend/src/app/components/ui/liquid-surface.ts` over raw palette values,
custom hex colors, or copied shadow strings.

When changing layout or spacing, update the corresponding loading state at the
same time. Table skeleton helpers (`SkeletonLine` and `SkeletonDot`) live in
`frontend/src/app/components/shared/TablePrimitive.tsx`; the shared full-screen
gate is `frontend/src/app/components/shared/FullScreenLoader.tsx`. Many feature
loading states are colocated with their component, so search for
`animate-pulse`, `SkeletonLine`, and `SkeletonDot` in the affected feature.

Preserve the accessibility baseline:

- Every interactive element needs a visible focus indicator.
- Icon-only buttons need an accessible name, normally `aria-label`.
- Non-submit buttons inside forms need `type="button"`.
- Selection and toggle state must be represented with the appropriate ARIA
  attribute, not color alone.
- Use a native checkbox with `TABLE_CHECKBOX_CLASS` from
  `TablePrimitive.tsx` for standalone checkbox inputs. `CheckSquare` is the
  directory/picker row selection visual and is decorative by default.

## Frontend Structure

- Route and page components live in `frontend/src/app/`.
- Shared domain types live in
  `frontend/src/app/components/shared/types.ts`.
- Calls to the Express backend belong in `frontend/src/app/lib/mikeApi.ts` so
  authentication, API error parsing, and request behavior stay consistent.
- Reusable client behavior belongs in `frontend/src/app/hooks/` or
  `frontend/src/app/lib/`, with a colocated `*.test.ts` or `*.test.tsx` file.
- Use the `@/` alias for imports rooted at `frontend/src/`.

Do not expose raw backend, database, provider, or stack errors in the UI. Map
known 4xx responses to intentional messages and use the generic fallback
helpers in `frontend/src/app/lib/userFacingError.ts` for unexpected failures.

## Backend Structure

- `backend/src/app.ts` configures Express, middleware, rate limits, and route
  mounting.
- HTTP handlers live in `backend/src/routes/`.
- Reusable domain and infrastructure logic lives in `backend/src/lib/`.
- Authentication and other request middleware live in
  `backend/src/middleware/`.
- LLM provider creation is centralized in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-legal-products/mike](https://github.com/open-legal-products/mike) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
