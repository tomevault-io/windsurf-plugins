---
trigger: always_on
description: Guidance for any coding agent working in this repository — the single, tool-agnostic source of truth for how Noteside is built. Keep it accurate as the architecture changes.
---

# AGENTS.md

Guidance for any coding agent working in this repository — the single, tool-agnostic source of truth for how Noteside is built. Keep it accurate as the architecture changes.

## What this is

Noteside — an offline & **keyboard-first** notes desktop app (full vim _and_ conventional shortcuts, both first-class), plus its marketing site. Turborepo + pnpm monorepo:

- `apps/desktop` — Tauri 2 + React 19 + Vite + TypeScript (the app)
- `apps/landing` — Vite + React 19 + TypeScript + Tailwind v4 (the site)
- `apps/docs` — Fumadocs on React Router 7 (SPA, prerendered) — the documentation site at **docs.noteside.app** (`:3002` via `pnpm dev:docs`)
- `apps/brand` — Vite + React + Tailwind, the standalone **brand guide** (internal reference only; not deployed, not linked from the landing). Runs on `:3001` via `pnpm dev:brand`; has its own `logo.tsx` copy + `styles.css` (the brand CSS).

## Commands

Run from the repo root (Turborepo fans out to the right package):

```bash
pnpm install
pnpm dev              # everything: landing (:3000) + desktop Tauri window
pnpm dev:landing      # landing only
pnpm dev:desktop      # desktop only — runs `tauri dev` (needs Rust + Tauri system deps)
pnpm typecheck        # tsc --noEmit across the workspace  ← primary verification gate
pnpm build            # web bundles for all apps (Vite; docs via react-router)
pnpm lint             # oxlint (root; config .oxlintrc.json)   — pnpm lint:fix to autofix
pnpm format           # oxfmt (writes in place; root; config .oxfmtrc.json) — pnpm format:check to verify
```

Desktop / Tauri specifics:

```bash
pnpm --filter @noteside/desktop dev:web     # Vite only, no native window (browser dev)
pnpm --filter @noteside/desktop build       # web bundle → apps/desktop/dist
pnpm tauri build                            # full native bundle (see Icons below)
cd apps/desktop/src-tauri && cargo check    # verify the Rust shell compiles
# icon source of truth = scripts/mark.html (brand mark in Newsreader, exact design CSS).
# Render it with headless Chrome (see the file header) → src-tauri/app-icon.png, then:
pnpm --filter @noteside/desktop tauri icon src-tauri/app-icon.png  # → all bundle icons (.icns/.ico/png)
pnpm demo:build                             # build desktop web bundle → apps/landing/public/demo
pnpm --filter @noteside/desktop bench       # vitest benchmarks — JS hot paths (computeBacklinks, parse)
cd apps/desktop/src-tauri && cargo bench    # criterion — search/scan at 1k/10k/50k (benches/perf.rs)
```

**Performance posture (measured — see `benches/perf.rs` + `src/perf.bench.ts`).** `fuzzy_files` is sub-ms even at 50k (nucleo — untouched). `content_search` is a fast in-memory scan — even a rare/no-match query (the worst case) is ~2ms @1k, ~20ms @10k, ~100ms @50k, far past realistic note counts; no DB or index to build or sync. `scan_notebook` is a one-time 156ms @10k. The **sidebar virtualizes** above 100 notes (`@tanstack/react-virtual`; the plain list renders verbatim below). **Backlinks** run as a Rust command (`commands::backlinks` → `links.rs`, mirrors `src/links.ts`) so the scan stays off the JS thread and only references cross IPC.

Lint/format are oxc, run from the repo root (not per-package, not through Turbo): `oxlint` (`.oxlintrc.json`, correctness=error/suspicious=warn, react+typescript+import plugins) and `oxfmt` (`.oxfmtrc.json`; pinned exactly — still beta). The codebase is kept oxfmt-formatted, **including `styles.css`** (so it's no longer byte-identical to the design source — diff semantically, not line-for-line). Tests: **Vitest** (`pnpm test`, node env — pure modules only, no jsdom/CM6) covers `autosave.ts` (incl. the cross-note regression), `settings.ts` round-trip, and the mock backend; **cargo test** (`pnpm test:rust`) covers `notebook`/`search` (`#[cfg(test)]` modules). No e2e yet (deferred). `pnpm typecheck` + `pnpm lint` + both test suites are the gates.

## Architecture: the parts that span files

**The editor is CodeMirror 6 + `@replit/codemirror-vim`** (`apps/desktop/src/editor/`), not a bespoke engine — the old `src/vim.ts` reducer is gone. Ex-commands, the `<Space>` leader palette, `gf`, and `:`-actions dispatch through a module-level handler registry: `ex-commands.ts` (`Vim.defineEx`/`Vim.map` + `setActiveHandlers`) → `app.tsx`. To follow any save/quit/open/follow flow, read `editor/editor.tsx` (mount + handler wiring) + `editor/ex-commands.ts` (where actions/ex-commands originate) + `app.tsx` (the handlers) together. `live-preview.ts`/`wikilinks.ts` add reveal-on-cursor-line decorations; `theme.ts` maps the CSS-var design tokens into the editor.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshxfi/noteside](https://github.com/joshxfi/noteside) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
