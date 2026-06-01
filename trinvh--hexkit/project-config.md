---
trigger: always_on
description: Project guidance for Claude Code working in this repo. Read this before making
---

# CLAUDE.md

Project guidance for Claude Code working in this repo. Read this before making
changes. (User-level rules in `~/.claude/` still apply; this file is the
project-specific layer.)

## What this is

**Hexkit** — a fast, **offline** cross-platform desktop developer toolbox
(~38 tools) built with **Tauri 2** (Rust core + system WebView) and **React 19**.
All tool logic lives in pure Rust; the UI is a thin client. Nothing the user
pastes ever leaves the machine.

## Related working directories (siblings, not subdirs)

Two sibling repos live next to this one and are commonly added to the same
session via `/add-dir`. Treat them as separate projects with their own
package managers and lockfiles — do not vendor them into this repo.

| Path                          | What it is                                                            |
| ----------------------------- | --------------------------------------------------------------------- |
| `../hexkit-devutils-landing`  | Marketing site for hexkit.app. Vite + React 19 + Tailwind v4, deployed to Cloudflare Workers with Static Assets. Worker reserves `/api/*` for future license endpoints. `make help` lists tasks. |
| `../hexkit-devutils-raycast`  | Raycast extension. Renders text-in/text-out tools inline by shelling out to the `hexkit` CLI; falls back to `hexkit://<action>?input=…` deep links for visual tools (color, QR, certs, diff). Tool catalog in `src/lib/tools.ts`; an `inline` flag toggles each tool's path. `npm run build` / `npm run dev`. |

When working across them:

- **Action ids and `hexkit://` deep links are the contract** between this
  repo and the Raycast extension. If you rename an action in
  `crates/devtools-core/src/actions.rs`, update `src/lib/tools.ts` in the
  Raycast repo to match.
- **The CLI surface is also part of that contract.** Inline Raycast
  commands shell out via `hexkit <action> '<json-params>'`; keep that
  signature stable (see `crates/hexkit-cli/src/lib.rs`).
- **Brand tokens live in this repo's `src/styles/globals.css`.** The
  landing site mirrors them in its own `src/index.css` `@theme` block —
  keep the two in sync when adjusting the palette.

## Architecture (the one rule that matters most)

**All tool logic lives in Rust, in `crates/devtools-core`.** The frontend never
implements tool logic — it calls the backend.

```
crates/devtools-core/   Pure, Tauri-independent logic. One module per tool, each
                        with a `dispatch(action, params) -> ToolResult<Value>` fn.
                        Routed by src/actions.rs `run(action, params)`.
crates/hexkit-cli/      Headless `hexkit` binary over the same dispatcher.
src-tauri/              Tauri shell: the single `run_action` command, clipboard,
                        deep link (`hexkit://`). Crate = `hexkit`, lib = `hexkit_lib`.
src/                    React UI: tool registry, shared primitives, per-tool views.
```

Every tool is reachable identically from the desktop app, the CLI, and
`hexkit://` deep links because they all route through `devtools_core::run`.

- Errors: return `ToolError` (`thiserror`, serialized `{kind, message}`).
- **Dispatch must propagate with `?`:** `to_value(convert(&p.input)?)`, never
  `to_value(convert(&p.input))` — the latter serializes the `Result` as
  `{"Ok": …}` and the UI silently shows nothing. (This was a real bug.)

## Frontend conventions

- **Tool registry** (`src/tools/registry.ts`) drives the sidebar, palette, and
  routing. Tool components are **lazy-loaded** (`React.lazy`) so each is its own
  chunk; CodeMirror language grammars are dynamically imported too.
- **One folder per tool** under `src/tools/<name>/`: `api.ts` (typed
  `runAction` wrappers), `run.ts` (mode options + a `run*` fn returning a
  `Promise | null`), `<Name>Tool.tsx`, and tests.
- **Shared IPC:** `runAction<T>(action, params)` in `src/lib/ipc.ts`. Live,
  debounced, race-cancelling calls via `useLiveAction`.
- **Shared UI primitives** (`src/components/ui/`): `TransformLayout` (input→output
  with toolbar, `outputFooter`, Paste/Sample/Clear), `ResultLayout` + `ResultList`
  (labelled rows), `CodeEditor` (CodeMirror, mocked as a `<textarea>` in tests),
  `Segmented`, `Toggle`, `TextField`, `CopyButton`, `Menu` (portal context menu),
  `InputActions` (Paste/Sample/Clear), `JsonView` (inline JSON highlighter).
  Reuse these; don't hand-roll layouts.
- **Per-tab persisted state:** use **`useToolState(field, initial)`** (a
  `useState` drop-in scoped to the active tab via `TabContext`) for inputs and
  option selections — NOT for derived outputs (those recompute via
  `useLiveAction`). State persists across tab switches and app restarts.
- **Tabs/store:** Zustand store `src/store/app.ts` (persisted to localStorage key
  `hexkit:app`: `tabs`, `activeTabId`, `tabState`, `pinned`, `recents`, collapse
  flags). `activeToolId` mirrors the active tab. `useSeed` carries one-shot
  prefill for smart-detect.

## Adding a tool (the standard flow)

1. **Rust first, test-first.** New module in `crates/devtools-core/src/<name>.rs`
   with a pure fn + `dispatch`; route it in `actions.rs` and declare it in
   `lib.rs`. Write unit tests (happy path, edge cases, error cases) and confirm
   RED before implementing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trinvh/hexkit](https://github.com/trinvh/hexkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
