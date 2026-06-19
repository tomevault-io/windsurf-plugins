---
trigger: always_on
description: - `apps/web/`: React + Vite admin dashboard.
---

# Repository Guidelines

## Project Structure & Module Organization
- `apps/web/`: React + Vite admin dashboard.
- `apps/voice-gateway/`: narrow Node.js runtime for Twilio Voice, Media Streams, and OpenAI Realtime.
- `convex/`: primary backend and source of truth for auth, business state, booking, knowledge, and workflows.
- `packages/`: shared TypeScript libraries (`ai`, `config`, `domain`, `providers`, `shared`, `telemetry`, `testing`).
- `docs/`: architecture notes, ADRs, provider docs, and Linear tracking docs.
- `docker/` and `scripts/`: local ops and seed helpers.

Do not edit generated files in `convex/_generated/` by hand.

## Build, Test, and Development Commands
- `pnpm install`: install workspace dependencies.
- `pnpm convex dev`: start Convex dev, generate backend types, and sync components.
- `pnpm dev`: run Convex, web, and voice gateway together.
- `pnpm build`: build every workspace package and app.
- `pnpm typecheck`: run TypeScript checks across the monorepo.
- `pnpm test`: run all Vitest suites.
- `pnpm seed:demo`: seed demo tenants and sample data.

## Coding Style & Naming Conventions
- TypeScript ESM throughout; use 2-space indentation and LF line endings per `.editorconfig`.
- Use `PascalCase` for React components, `camelCase` for functions/utilities, and descriptive domain folders under `convex/` (for example `convex/appointments/booking.ts`).
- Keep voice personalization snapshot-based: structured business facts stay authoritative in Convex; RAG augments documents and FAQs only.
- Prefer `rg` for search. Use `apply_patch` for manual edits. Add TSDoc/JSDoc only for exported or non-obvious modules.

## Design System Guidelines
- Treat `shadcn/ui` as the default UI system for `apps/web`. Prefer composing existing shadcn primitives over custom wrappers when the primitive already solves the problem well.
- Preserve shadcn component structure, variants, accessibility, and interaction behavior. Do not restyle component internals unless the change is clearly about spacing or consistency.
- Use Geist Sans as the primary sans-serif font across the web app.
- Use Lucide as the only icon set in `apps/web`. Do not introduce Tabler, Heroicons, or mixed icon families for operator UI work.
- Keep the UI on a 4px base grid with an 8px default major rhythm.
- Use this spacing scale for layout and component spacing: `4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 96, 128`.
- Prefer Tailwind spacing utilities from that scale. Avoid arbitrary spacing values like `p-[3px]`, `top-[0.3rem]`, or one-off offsets unless geometry or animation truly requires them.
- Use explicit radius tokens. Avoid bare `rounded` in `apps/web`; pick a named size or an allowed documented exception.
- Default radius tier in `apps/web`:
  - `rounded-xl` for standard operator surfaces and controls, including cards, dialogs, drawers, tables, settings sections, inputs, selects, menu items, search fields, and row actions
- Allowed radius exceptions:
  - `rounded-full` for pill buttons, pill navigation such as settings tabs, badges, avatars, toggles, progress bars, slider thumbs, and circular icon wells
  - `2px` to `4px` only for microscopic geometry such as checkbox corners, chart swatches, and tooltip arrows
  - asymmetric `16px` message bubbles for threaded conversation UI and transcript/chat surfaces
- Normalize spacing decisions across the app:
  - icon gap: `8`
  - button horizontal padding: `16`
  - button vertical padding: `12`
  - input padding: `12` or `16`
  - card padding: `24`
  - internal card gaps: `12` or `16`
  - section spacing: `64` or `96`
- Keep page-level rhythm consistent across dashboard surfaces. Similar pages should use the same top spacing, bottom spacing, section gaps, and card gutters unless there is a clear product reason not to.
- When adjusting existing UI, prioritize consistency over pixel-perfect preservation of older bespoke spacing.
- Do not redesign pages during spacing cleanup. Preserve colors, typography choices, copy, logic, responsiveness, and route architecture unless the task explicitly asks for a broader visual change.

## Localization Guidelines
- Use translation keys for new dashboard UI copy under `apps/web/`; do not add new hardcoded user-facing English strings when the text belongs in the operator UI.
- Keep locale files in `apps/web/public/locales/{lng}/{ns}.json` and group keys by feature/intent rather than by component implementation details.
- Avoid string concatenation in translated sentences. Prefer interpolation through the translation layer instead.
- Keep translated text separate from date, time, and number formatting. Use the active locale with `Intl` or Luxon for formatting instead of baking localized formats into translation strings.

## Testing Guidelines
- Vitest is the default test runner.
- Name tests `*.test.ts` or `*.test.tsx` and keep them close to the code they cover.
- Prioritize coverage for booking logic, snapshot generation, authz helpers, webhook handling, and telemetry redaction.
- For Convex behavior, follow the official Convex testing guidance and prefer `convex-test` with the real schema over ad hoc mocked `ctx.db` chains.
- Put Convex-backed regression tests in `convex/tests/`. Keep pure helper tests next to the `convex/` module they cover.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lobbystack/lobbystack](https://github.com/lobbystack/lobbystack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
