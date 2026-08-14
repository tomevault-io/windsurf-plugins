---
trigger: always_on
description: transforms in `lib/` and call them from the store (see `app-store.ts` ↔
---

# CLAUDE.md

Guidance for AI agents working in this repo. Keep it lean — it points at the
code and docs rather than duplicating them. When this file and the code
disagree, **trust the code**.

## What this is

**Swarmterm** — a desktop multi-terminal app: a left navbar of workspaces, each
a binary split-tree of real terminal panes (xterm.js front, `portable-pty`
shell back), plus an optional Chrome-style web-preview column. Built on
**Tauri 2 + Rust** (frontend: **React 19 + TypeScript + Vite**). Ported from an
earlier Electron app; no state is persisted between launches by design.

`README.md` is a short user-facing intro (the pitch, install, first run) and
`docs/user-guide.md` is the detailed end-user guide — both English, no code
guidance. The manual smoke-test checklist is `docs/manual-smoke-tests.md`.
Update them when you ship user-visible behaviour: new features go into the
guide (and README only if they change one of its three core ideas).

**Cross-platform intent:** build for macOS and Linux as well as Windows, even
though primary development happens on Windows. Don't hard-code Windows-only
assumptions; gate platform code behind `#[cfg(...)]` (Rust) or runtime checks (TS).

## Commands

Run from the repo root unless noted.

| Command | What it does |
|---|---|
| `npm install` | Install JS dependencies. |
| `npm run tauri dev` | Run the full app (Vite HMR frontend + Rust auto-rebuild). |
| `npm test` | Run the JS/TS unit suite (Vitest, run-once). |
| `npm run test:watch` | Vitest in watch mode. |
| `npx tsc --noEmit` | Type-check the whole frontend (strict). |
| `npm run build` | Frontend-only build (`tsc && vite build`) — no Rust. |
| `npm run logo` | Regenerate every logo asset from `scripts/gen-logo.mjs` (also runs `tauri icon`). |
| `npm run tauri build` | Production bundle (installer). |
| `npm run tauri build -- --no-bundle` | Release binary, skip installer. |
| `cargo test` *(from `src-tauri/`)* | Rust unit tests (pty/shell helpers). |

**Before claiming done:** run `npm test`, `npx tsc --noEmit`, and — if you
touched `src-tauri/` — `cargo test`. Don't assert success without the output.

## Architecture

```
┌─ Renderer (src/) ─────────────┐         ┌─ Backend (src-tauri/src/) ─┐
│ components/  React UI          │         │ lib.rs     builder/plugins │
│ store/       zustand state     │ invoke  │ commands.rs #[command] fns │
│ lib/         pure logic (TDD)  │ ──────► │ pty.rs      spawn + reader │
│ tauri/       IPC bridge ───────┼─────────┤ shell.rs    shell discovery│
│                                │ Channel │ tray.rs                    │
└────────────────────────────────┘ ◄────── └────────────────────────────┘
                                    PtyOut
```

- **`src/tauri/*` is the ONLY IPC surface.** There is no `window.api` shim. Every
  call into Rust goes through a thin typed module here: `terminal.ts` (pty
  create/write/resize/kill + the `Channel<PtyOut>` stream), `window.ts`,
  `dialog.ts`, `clipboard.ts`, `shell.ts`, `preview.ts`, `popout.ts`. New
  backend calls get a new function in one of these — components never call
  `invoke` directly.
- **`#[tauri::command]` handlers live in `commands.rs`** and delegate to module
  logic (`pty.rs`, `shell.rs`). Register new commands in the
  `invoke_handler!` list in `lib.rs`.
- **Per-terminal streaming:** each pty gets its own `Channel<PtyOut>`; a Rust
  reader thread decodes output and sends `Data` chunks, then a final `Exit`.
  The Serde tagging (`type`/`payload`, camelCase) must stay in lockstep with the
  `PtyOut` union in `src/tauri/terminal.ts`.

## Module boundaries (respect these)

- **`src/lib/`** — pure, framework-free logic, each with a `*.test.ts` beside it
  (layout-tree, web-url, templates, terminal-session, appearance, etc.). This is
  where business rules go so they can be unit-tested without a DOM or a pty.
- **`src/store/`** — zustand stores. UI state and actions only; keep pure
  transforms in `lib/` and call them from the store (see `app-store.ts` ↔
  `layout-tree.ts`).
- **`src/lib/terminal-registry.ts`** — owns the live xterm `Terminal` instances
  **outside React's render tree**, keyed by `terminalId`. Components attach/detach
  by id; the registry survives remounts so a pane re-parenting (split collapse)
  doesn't kill the shell. Don't put xterm instances in React state.
- **`src/components/`** — thin. Layout + wiring; logic belongs in `lib`/`store`.
  `components/ui/` is shadcn-style primitives (button, dropdown-menu).

## Conventions

- **Imports:** use the `@/` alias for `src/` (`@/lib/...`), configured in both
  `vite.config.ts` and `tsconfig.json`.
- **TypeScript is strict**, including `noUnusedLocals` / `noUnusedParameters` /
  `noFallthroughCasesInSwitch`. Dead code fails the type-check.
- **TDD for `lib/`:** write/extend the `*.test.ts` first. Run via Vitest.
- **Comments explain *why*, not *what*.** This codebase leans on dense rationale
  comments for non-obvious platform/lifecycle decisions — match that density when
  you add similar logic (see `pty.rs`, `terminal-registry.ts` for the bar).
- **Styling:** Tailwind (`tailwind.config.cjs`) + CSS variables in
  `src/index.css` for light/dark theming. The visual target is **VS Code** —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duongducnguyen/swarmterm](https://github.com/duongducnguyen/swarmterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
