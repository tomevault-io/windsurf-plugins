---
trigger: always_on
description: **CRITICAL:** Before declaring victory on any task and before committing to git, the following four commands must pass with NO WARNINGS:
---

# Repository Guidelines

## Pre-Commit Checklist

**CRITICAL:** Before declaring victory on any task and before committing to git, the following four commands must pass with NO WARNINGS:

1. `pnpm lint` — Must pass with zero warnings
2. `pnpm fmt:check` — Must pass with zero warnings
3. `pnpm check-types` — Must pass with zero warnings
3. `pnpm test` — Must pass with zero warnings

Do not proceed with commits until all four checks are clean.

## Commands

### Development
- `pnpm dev` — Start Next.js dev server at http://localhost:3000
- `pnpm build` — Compile production bundle
- `pnpm check-types` — Run `tsc --noEmit` for type diagnostics

### Linting & Formatting
- `pnpm lint` — Run oxlint lint
- `pnpm fmt` — Apply oxfmt formatting

### Testing
- `pnpm test` — Run all tests once
- `pnpm test path/to/file.test.ts` — Run a single test file
- `pnpm test -t "test name"` — Run tests matching a pattern
- `pnpm test:coverage` — Run tests with coverage report

### Database
- `pnpm db:generate` — Generate Drizzle migrations
- `pnpm db:push` — Push schema to database
- `pnpm db:migrate` — Apply migrations
- `pnpm db:studio` — Open Drizzle Studio

## Code Style

### General
- TypeScript only, `strict` enabled
- 2-space indentation (oxfmt enforces)
- Prefer small, pure modules
- Node.js >= 24 required

### Naming Conventions
- **Files/folders:** kebab-case (`user-settings.ts`)
- **React components:** PascalCase exports (`UserSettings`)
- **Helpers/hooks:** camelCase named exports (`useUserSettings`)

### Imports
- Use `@/...` path aliases for app-specific imports
- Import shared UI components from `@domainstack/ui/*` (e.g., `@domainstack/ui/button`)
- oxfmt auto-organizes imports on save
- Client components must start with `"use client"`

### Types
- Shared domain types in `@domainstack/types` package
- Enum const arrays (primitives) in `@domainstack/constants` (Drizzle pgEnums derive from these)
- Do NOT use Zod for simple enums or internal database types
- Import types from `@domainstack/types`

### Tailwind Classes
- oxfmt enforces sorted Tailwind classes via `useSortedClasses` rule
- Use `cn()` from `@domainstack/ui/utils` for conditional classes

## Web Interface Guidelines

Concise rules for building accessible, fast, delightful UIs. Use MUST/SHOULD/NEVER to guide decisions.

### Interactions

#### Keyboard

- MUST: Full keyboard support per [WAI-ARIA APG](https://www.w3.org/WAI/ARIA/apg/patterns/)
- MUST: Visible focus rings (`:focus-visible`; group with `:focus-within`)
- MUST: Manage focus (trap, move, return) per APG patterns
- NEVER: `outline: none` without visible focus replacement

#### Targets & Input

- MUST: Hit target ≥24px (mobile ≥44px); if visual <24px, expand hit area
- MUST: Mobile `<input>` font-size ≥16px to prevent iOS zoom
- NEVER: Disable browser zoom (`user-scalable=no`, `maximum-scale=1`)
- MUST: `touch-action: manipulation` to prevent double-tap zoom
- SHOULD: Set `-webkit-tap-highlight-color` to match design

#### Forms

- MUST: Hydration-safe inputs (no lost focus/value)
- NEVER: Block paste in `<input>`/`<textarea>`
- MUST: Loading buttons show spinner and keep original label
- MUST: Enter submits focused input; in `<textarea>`, ⌘/Ctrl+Enter submits
- MUST: Keep submit enabled until request starts; then disable with spinner
- MUST: Accept free text, validate after—don't block typing
- MUST: Allow incomplete form submission to surface validation
- MUST: Errors inline next to fields; on submit, focus first error
- MUST: `autocomplete` + meaningful `name`; correct `type` and `inputmode`
- SHOULD: Disable spellcheck for emails/codes/usernames
- SHOULD: Placeholders end with `…` and show example pattern
- MUST: Warn on unsaved changes before navigation
- MUST: Compatible with password managers & 2FA; allow pasting codes
- MUST: Trim values to handle text expansion trailing spaces
- MUST: No dead zones on checkboxes/radios; label+control share one hit target

#### State & Navigation

- MUST: URL reflects state (deep-link filters/tabs/pagination/expanded panels)
- MUST: Back/Forward restores scroll position
- MUST: Links use `<a>`/`<Link>` for navigation (support Cmd/Ctrl/middle-click)
- NEVER: Use `<div onClick>` for navigation

#### Feedback

- SHOULD: Optimistic UI; reconcile on response; on failure rollback or offer Undo
- MUST: Confirm destructive actions or provide Undo window
- MUST: Use polite `aria-live` for toasts/inline validation
- SHOULD: Ellipsis (`…`) for options opening follow-ups ("Rename…") and loading states ("Loading…")

#### Touch & Drag

- MUST: Generous targets, clear affordances; avoid finicky interactions
- MUST: Delay first tooltip; subsequent peers instant
- MUST: `overscroll-behavior: contain` in modals/drawers
- MUST: During drag, disable text selection and set `inert` on dragged elements
- MUST: If it looks clickable, it must be clickable

#### Autofocus

- SHOULD: Autofocus on desktop with single primary input; rarely on mobile

### Animation

- MUST: Honor `prefers-reduced-motion` (provide reduced variant or disable)
- SHOULD: Prefer CSS > Web Animations API > JS libraries
- MUST: Animate compositor-friendly props (`transform`, `opacity`) only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jakejarvis/domainstack.io](https://github.com/jakejarvis/domainstack.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
