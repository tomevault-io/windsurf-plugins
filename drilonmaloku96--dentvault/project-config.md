---
trigger: always_on
description: A cross-platform desktop dental patient management app for independent practitioners and small clinics.
---

# DentVault — Project Guide for Claude

## What is DentVault?

A cross-platform desktop dental patient management app for independent practitioners and small clinics.
Built with **Tauri 2 + SvelteKit + Svelte 5 + TypeScript + SQLite + Tailwind CSS v4 + shadcn-svelte**.

**Core principle**: Every piece of clinical data must be structured, tagged, and queryable — not buried in free-text notes. The app's primary long-term value is clinical outcome tracking and statistical analysis.

**Reference docs** (read these when working on specific areas):
- `docs/claude/DESIGN_SYSTEM.md` — **Design system specification**: color palette (warm paper + clinical teal + semantic colors), typography scale, component library, dark mode, layout patterns. Defines all Tailwind color tokens used throughout the app.
- `docs/claude/DATA_INTEGRITY.md` — **MANDATORY before adding/changing any feature**: dataset & evaluation mindset, known mistake patterns (dead pipelines, string-matched queries, notation mixing, enum drift, rate denominators), pre-merge checklist
- `docs/claude/DENTAL_CHART.md` — watch status, root canals, crown findings, bridge/prosthesis, surface picker
- `docs/claude/TIMELINE.md` — timeline entries, rich text editor, ortho snapshots, plan indicators
- `docs/claude/SCHEDULE.md` — calendar pointer system, appointment & block interactions
- `docs/claude/FEATURES.md` — vault/files, settings page, customizable systems, patient form

---

## Technical Conventions

- **Svelte 5 runes**: `$state()`, `$derived()`, `$effect()`, `$props()`
- **Snippet slots**: `{@render children()}`
- **Tailwind v4**: `@theme inline` blocks in `src/app.css`, no `tailwind.config.js`
- **Colors**: hex-based CSS custom properties in `src/app.css` (see `DESIGN_SYSTEM.md` for the complete palette: warm paper, deep-pine sidebar, clinical teal primary, semantic colors for critical/warning/success/info, and procedure-type colors for endodontics/periodontics/orthodontics/prosthetics/hygiene). All components inherit from tokens—no hardcoded colors.
- **Theme switching**: `theme.svelte.ts` must set **all three** of `.dark` class (drives Tailwind `dark:` variants), `data-theme` attribute (drives the `:root[data-theme]` token override blocks in `app.css` — without it the `@media (prefers-color-scheme: dark)` block keeps every token dark when the OS is dark, so "light mode" only changed the few class-driven bits), and `style.colorScheme` (native form controls/scrollbars).
- **shadcn-svelte**: components at `$lib/components/ui/`, install with `npx shadcn-svelte@1.1.1 add <name> -y`
- **DB access**: exclusively through `src/lib/services/db.ts`, positional `$1, $2` params
- **`db.ts` is now a barrel** (`src/lib/services/db.ts` → `export * from './db-local'; export * from './db-core';`) — a Phase 0 step of `ROADMAP_MULTI_COMPUTER.md`. `db-local.ts` holds `runMigrations` and `getDb()` (solo-mode transport: local SQLite via tauri-plugin-sql). `db-core.ts` holds the 158 data functions, written against the `DataTransport` interface (`db-transport.ts`: `{select, execute}`) rather than the concrete `Database` class, so a future `db-remote.ts` (server transport) can be swapped in without touching db-core. All existing `import ... from '$lib/services/db'` call sites are unaffected — keep importing from `db.ts`, not the split files directly.
- **Migrations**: the DDL itself lives in `shared/schema-statements.json` (a `{latestVersion, statements: [{version, sql}]}` JSON file at the repo root, **not** inside `src/`) — `db-local.ts` imports it directly (`resolveJsonModule`), and `dentvault-server` (Rust) reads the identical file, so solo mode and connected mode apply byte-identical migrations with zero porting risk. Append new `{version, sql}` entries to the JSON array; **never modify an existing entry.** Update `latestVersion` after adding. Current version: **69**. Test every new migration's SQL against a copy of a real vault DB with the `sqlite3` CLI before shipping — SQLite rejects some common syntax (e.g. derived-table column-list aliases `AS t(a,b)`, which made v65 a permanent no-op until fixed). `shared/` sits outside SvelteKit's default dev-server `fs.allow` boundary (`src`/`node_modules`/`.svelte-kit` only) — `vite.config.ts` explicitly allows the project root (`server.fs.allow: ['.']`) so `npm run tauri dev` can read it; this only affects `dev`, not `npm run build`, which has no such middleware.
- **`getDb()` caches the handle only after `runMigrations` succeeds** — never assign `db` before migrations finish. The old order (assign, then migrate) swallowed the first migration error and served the unmigrated DB forever after, silently freezing vaults at an old schema version with no visible error.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drilonmaloku96/DentVault](https://github.com/drilonmaloku96/DentVault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
