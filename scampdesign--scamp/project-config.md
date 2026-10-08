---
trigger: always_on
description: This file is read by Claude Code on every session. Follow everything here without being asked.
---

# CLAUDE.md — Scamp App

This file is read by Claude Code on every session. Follow everything here without being asked.

---

## Project Overview

Scamp is a local-first Electron design tool. Users draw rectangles on a canvas and the app auto-saves real TSX + CSS Module files. Bidirectional sync means external edits (agent or manual) reload the canvas. The stack is Electron + electron-vite + React + TypeScript + Zustand.

See `prd-scamp-poc.md` for full product context.

---

## Where things are

Read this before grepping. Everything below is a starting point, not the
whole story — but starting in the wrong file is the most expensive thing
you can do here, and a wrong guess costs more context than this table.

| Task | Start at |
|---|---|
| Read a web page into the model | `src/shared/captureScript.ts` (runs IN the page), `src/shared/importCapture.ts` (the contract + `CAPTURED_PROPERTIES`) |
| Change what an import produces | `src/renderer/lib/importReduce.ts` — the pure reducer |
| The import's report of what it lost | `src/renderer/lib/importReport.ts` |
| Import fonts, theme tokens, images | `importFonts.ts`, `importTokens.ts`, then `src/renderer/src/components/projectShell/useWebsiteImport.ts` for the orchestration |
| Measure import fidelity | `scripts/import-fidelity.mjs` (geometry), `scripts/import-pixels.mjs` (pixels), findings in `docs/notes/import-parity-log.md` |
| TSX / CSS output | `src/renderer/lib/generateCode/` — `declarations.ts` is where a property becomes CSS |
| Read files back into the model | `src/renderer/lib/parseCode/` |
| A CSS property's typed mapping | `src/renderer/lib/cssPropertyMap.ts`, `parsers/` for each shorthand |
| Draw an element on the canvas | `src/renderer/src/canvas/ElementRenderer.tsx` |
| Canvas frame, zoom, theme variables | `src/renderer/src/canvas/Viewport.tsx` |
| Theme tokens reaching the canvas | `src/renderer/store/canvas/slices/designSystem.ts` (`setThemeData`), consumed in `Viewport.tsx` |
| Canvas state, selection, elements | `src/renderer/store/canvas/slices/` — one slice per domain |
| A properties-panel control | `src/renderer/src/components/sections/` — one file per section |
| Pages, views, routes on disk | `src/main/ipc/componentOps.ts` |
| Start-screen thumbnails | `src/renderer/src/lib/projectThumbnail.ts`, capture in `exportCapture.ts` |
| An IPC channel | name in `src/shared/ipcChannels.ts`, handler in `src/main/ipc/<domain>.ts`, surface in `src/preload/index.ts` |
| What an agent can do over MCP | `src/main/mcp/tools.ts`, answered in `src/renderer/src/syncBridge/mcpResponder.ts` |
| Auto-update and releases | `electron-builder.yml`, `.github/workflows/release.yml`, `docs/notes/auto-update.md` |

**Canvas / preview divergence** has its own harness: `test/e2e/parity/`.
Read `docs/notes/parity-harness.md` first — its oracle deliberately shares
no code with the thing it checks, and that rule is easy to break by
accident.

---

## Non-Negotiable Rules

- **Never write `any`** in TypeScript — use proper types or `unknown` with a type guard
- **Never skip tests** for anything in `src/renderer/lib/` — these are pure functions and must be fully tested
- **Never use `console.log` for debugging** — use the logger utility or remove before committing
- **Never hardcode IPC channel name strings** — always use constants from `src/shared/ipcChannels.ts`
- **Never read from disk in the renderer** — all file operations go through IPC
- **Never assume a case-sensitive filesystem.** Linux CI is case-sensitive; macOS APFS (the default for dev machines) is case-insensitive, so `fs.access("components/button/button.tsx")` resolves to the `Button` folder. Use directory listings + string comparison when a check depends on exact casing, and design code/tests to behave the same on both platforms.
- **In Playwright tests, use `ControlOrMeta+` not `Control+`** for keyboard shortcuts that should be cross-platform. CodeMirror's `Mod-*` bindings (and most macOS-aware app shortcuts) map to `Cmd` on macOS and `Ctrl` on Linux. A literal `Control+End` is a no-op on macOS and the next `keyboard.type` splices into wherever `editor.click()` left the cursor — usually mid-line, producing CSS the parser then rejects.

---

## Code Standards

### General:
- Prefer writing clear code and use inline comments sparingly
- If a comment would be more than ~2 lines, OR captures multi-file architectural context, move it to a new or existing file under `docs/notes/` and reference it inline (`// see docs/notes/<slug>.md`). Inline comments stay short and only encode local WHY (incidents, browser quirks, hidden invariants).
- When you make a code change that affects an existing `docs/notes/` file, update the note in the same commit. Stale notes are worse than no notes.

### TypeScript

- Strict mode is on — no exceptions
- Prefer `type` over `interface` unless you need declaration merging
- All function arguments and return types must be explicitly typed
- No implicit returns in functions that should return a value
- Use `const` by default; `let` only when reassignment is necessary
- Avoid `!` non-null assertions — handle nulls explicitly

```ts
// ✅
const getElement = (id: string): Element | null => {
  return elements[id] ?? null;
};

// ❌

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [scampdesign/scamp](https://github.com/scampdesign/scamp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
