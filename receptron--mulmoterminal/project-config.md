---
trigger: always_on
description: Working notes for AI coding agents in this repo. Human-facing docs (what it is,
---

# CLAUDE.md — mulmoterminal

Working notes for AI coding agents in this repo. Human-facing docs (what it is,
install, features, full API/architecture) live in **README.md** — read it for
anything not covered here.

## Stack & package manager
- TypeScript. Web UI: **Vue 3 (Composition API)** + Vite (`src/`). Backend:
  **Express** + **node-pty**, run via **tsx** (`server/`). Shared code in `common/`.
- Package manager: **yarn** (yarn.lock). Use `yarn add`; don't hand-edit package.json.

## Run after changes
- `yarn format` — Prettier. `.prettierignore` excludes `*.md`, so Markdown is not reformatted.
- `yarn lint` — ESLint.
- `yarn typecheck` — `vue-tsc -b`, and it covers the whole repo: the root `tsconfig.json`
  references all five projects (app, node, server, and the two spec ones — including the specs
  colocated under `server/` rather than in `test/`). Adding a project means adding it there too,
  or nothing type-checks it and CI will not tell you.
- `yarn build` — `vue-tsc -b && vite build`.
- `yarn test` — **Vitest** (`test/**/*.spec.ts`). Mock external APIs; tests must run without API keys.
- `yarn dev` — server + Vite together (local development).

### Import a component at module scope, never inside a test

`await import("…/Foo.vue")` inside an `it` (or a helper an `it` awaits) pulls the component's
whole module graph through the transform, and **the first test to reach it is billed that time
against `testTimeout`**. On this repo that was 2132ms of loading against an 18ms mount — so the
file's first test looked 100x slower than its siblings, and on a loaded runner it was the one
that crossed 15s and went red (#1314). The test was never the slow part.

Load it once at module scope instead — `const Foo = (await import("…/Foo.vue")).default;`, a
top-level await, or a plain static import. Collection has no per-test budget, so the same work
costs nothing there.

The exception is a module that must be evaluated AFTER a non-hoisted mock: `vi.doMock` and
`vi.resetModules` only take effect on a later import, so those specs (`codeBlockCopy.spec.ts`,
several under `test/server/`) keep the import inside the test on purpose. `vi.mock` is hoisted
and needs no such thing.

## No emojis
**Never use emojis anywhere in this project** — UI, source comments, docs, changelog, commit
messages, skills, CLI output. Icons are **Material Symbols (outlined)**, self-hosted via the
`material-symbols` npm package: `<span class="material-symbols-outlined">icon_name</span>`.
A global rule in `src/style.css` gives them `font-size: inherit`, so size them on the parent.

- A header button in config (`server/config/header-config.ts`) takes **`icon`**, not `emoji`.
  The `emoji` field still exists for end-user configs and wins over `icon` when both are set —
  don't use it in anything this repo ships.
- Three deliberate exceptions, all functional. Don't "fix" them:
  - `server/session/screen-rows.ts` — `/^\s*[❯›]\s/u` parses Claude Code's real terminal output.
  - `src/composables/useDynamicFavicon.ts` — the `❯` chevron drawn on canvas as the favicon mark.
  - `bin/mulmoterminal.js` — the CLI doctor's `✓ / ✗ / ○` (a terminal can't render an icon font).
- Compact status **notation** stays text, not icons: `⎇ main ●3 ↑2`, `●` unsaved dots, `−12` diff
  counts. Icons there are bigger and slower to scan.

## Layout
- `server/` — backend (PTY sessions, config, agents, backends). Ships user-facing skills in `server/skills/`.
- `src/` — Vue web UI (App.vue, components, composables, router).
- `common/` — code shared by server and UI. **Both** `tsconfig.server.json` and
  `tsconfig.app.json` include it, so a value or wire type that BOTH sides decide from
  (a shared config, an `/api/*` response shape, an enum) belongs here — never mirrored
  into `server/` and `src/` with a "keep the two copies in sync" comment. When the two
  sides genuinely differ, share the common core and keep each side's extras local, with
  a test pinning the asymmetry (see `common/sourceExtensions.ts` + its spec).
- `bin/` — CLI entry (`npx mulmoterminal`, `claude-ollama`, …).
- `docs/` — Jekyll site; bilingual guide under `docs/guide/{en,ja}` (keep both in sync).
- `plans/` — design notes per change. `test/` — Vitest specs.

## The grid has three view modes — read before changing anything a cell renders

`TerminalGrid.vue` is ONE `.stage` in three CSS states: the **tiled grid** (`!zoomed`), the
**cockpit roster** (`zoomed && listMode`, the default when you enlarge) and the **filmstrip**
(`zoomed && !listMode`). There is one component instance per cell and it is never remounted — the
enlarged one is **teleported** out, and in roster mode the rest are parked off-screen but **still
live**. The roster row is not a `TerminalCell` at all; it is a separate template with its own
chrome. And the tiled grid shows one page of ≤9 while both zoomed modes show **every** cell.

So "collapse the cell to its header" is already shipped in one mode, needs a new layout mechanism
in another, and lands on a different component in the third. Work it out from
[`docs/grid-view-modes.md`](docs/grid-view-modes.md) rather than from the screen you happen to be
looking at.

## MulmoClaude is the reference host — read it before wiring a shared package


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [receptron/mulmoterminal](https://github.com/receptron/mulmoterminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
