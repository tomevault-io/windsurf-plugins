---
trigger: always_on
description: - `--op-*` (navy `#0F172A` + teal `#0EA5A4` accent) is the canonical system
---

# Agent Context

## Token System
- `--op-*` (navy `#0F172A` + teal `#0EA5A4` accent) is the canonical system
- `--c-*` aliases maintained in `global.css` for backward compat
- `check-token-usage.mjs` enforces 0 `var(--c-*)` and 0 hardcoded hex violations
- All token migration complete — 0 VAR / 0 HEX violations

## Theme
- `ThemeContext.tsx` sets `data-theme="light"` explicitly (not `removeAttribute`)
- `[data-theme="night-shift"]` overrides use `var(--op-*)` references
- `@media (prefers-color-scheme: dark)` excludes `[data-theme="light"]`
- `--brand-lt` is teal `var(--op-accent-lt)`, not green

## Responsive Grids
- Utility classes in `components.css`:
  - `.grid-2/3/4/5` — equal-column grids, collapse to 1 col on mobile
  - `.grid-weighted` — preserves inline `gridTemplateColumns` on desktop, collapses to 1 col on mobile
  - `.layout-sidebar`, `.layout-sidebar--280`, `.grid-detail`, `.version-list-*`
- `Grid.tsx` uses `op-grid--{columns}` class
- `FormGrid` uses `op-grid--{columns}` class
- Mobile breakpoint: `!important` overrides via `@media (max-width: 767px)`
- Table-like weighted grids (header + rows) skip responsive treatment — horizontal scroll is acceptable

## TypeScript
- CrmForm uses `UseFormReturn<T, any, T>` for RHF 7.71 compatibility
- `unknown` values from `Record<string, unknown>` cast via `String()` when used as ReactNode

## Key Files — Architecture Infrastructure
- `frontend/shared/contexts/ThemeContext.tsx` — theme attribute logic
- `frontend/shared/styles/global.css` — token aliases + night-shift overrides
- `frontend/shared/styles/components.css` — grid utility classes
- `frontend/shared/ui-kit/styles.css` — `op-grid--*` classes + mobile overrides
- `scripts/check-token-usage.mjs` — CI guard

### Public Website (frontend/website)
- Static Astro 7 site, 6 locales (`en` at root, `fr/de/es/zh/ar` prefixed — `prefixDefaultLocale: false`). EN pages live at `src/pages/` root, other locales under `src/pages/[locale]/`; both wrap shared `src/components/pages/*Body.astro` via `PageShell`.
- Content is typed per-page in `src/i18n/content/{en,fr,de,es,zh,ar}.ts` (en is source of truth); `contentFor(locale)` assembles ui + content (falls back to en).
- NO `tsconfig.json` in the package on purpose — `typecheck-frontend.mjs` auto-skips it; use `tsconfig.typecheck.json` locally for `src/i18n` + `src/utils` checks.
- Hex colors only in `src/styles/tokens.css` (allowlisted in `check-token-usage.mjs`); `.astro` files are not scanned by the token guard.
- Blog uses Astro Content Layer (`src/content.config.ts`, `glob` loader — `type: "content"` is removed in Astro 7); posts keyed by `lang` in frontmatter.
- Social images: `pnpm run og` → `scripts/gen-og.mjs` (satori + sharp, static TTFs in `scripts/fonts/`, outputs `public/og/{locale}/home.png` + `og/en/logo.png`).

### Bridge Files (shared/lib)
- `commandBridge.ts` — Command registration events + types
- `searchBridge.ts` — Search provider registration + result types
- `objectTypeBridge.ts` — Object type registration + definition types
- `aiBridge.ts` — AI action registration + stream chunk types
- `relationshipBridge.ts` — Relationship type registration + record types
- `eventBus.ts` — Typed EventBus class, middleware, bridge adapter, EventRegistry
- `taskOrchestrator.ts` — DAG task execution with rollback + progress events
- `queryClient.ts` — TanStack Query client config + cache helpers
- `workspaceStorage.ts` — localStorage persistence helpers

### Pattern Components (shared/ui-kit/patterns)
- `DataGrid.tsx` — Virtualized grid with sort/filter/selection/resize
- `AIActionButton.tsx` — AI action button with streaming output + input prompt
- `RelationshipPanel.tsx` — Related objects list with link/unlink
- `RelationshipGraph.tsx` — SVG relationship graph (0 deps)

### Context Providers (shared/contexts)
- `AIInteractionContext.tsx` — AI action execution with SSE streaming
- `RelationshipRegistryContext.tsx` — Relationship CRUD via API
- `WorkspacePersistenceContext.tsx` — Workspace state save/restore (localStorage)
- `QueryProvider.tsx` — TanStack QueryClientProvider wrapper

## Work State

### Completed
- **GSC "Page with redirect" fix (2026-08-09)**: trailing-slash 301s from `frontend/website/nginx.conf` emitted *absolute* Location headers built from the internal http request (`/about` → 301 `http://opseron.com/about/` → 301 `https://opseron.com/about/`) — a scheme-downgrade chain flagged by Search Console. Added `absolute_redirect off;` + `server_name_in_redirect off;` + `port_in_redirect off;` → all `return 301` now emit relative `Location: /about/`, single hop, https preserved. Deploy: `docker compose build website && docker compose up -d website`, then re-run GSC "Validate fix".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Opseron/Opseron](https://github.com/Opseron/Opseron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
