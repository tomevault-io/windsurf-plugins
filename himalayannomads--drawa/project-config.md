---
trigger: always_on
description: Drawa is a browser canvas around coding agents: the Claude Code CLI, OpenCode for any other provider, and OpenAI's Codex CLI. `main.go` + `internal/` run their processes (one per session card, behind the backend registry in `internal/live`) and serve a JSON API. `web/` is a Vite + TypeScript frontend with no framework: plain DOM modules. Read `README.md` for how to run it, `CONTRIBUTING.md` for the development setup and an architecture overview, and `PRODUCT.md` for the design brief. This file c
---

# Guidelines for agents working on Drawa

Drawa is a browser canvas around coding agents: the Claude Code CLI, OpenCode for any other provider, and OpenAI's Codex CLI. `main.go` + `internal/` run their processes (one per session card, behind the backend registry in `internal/live`) and serve a JSON API. `web/` is a Vite + TypeScript frontend with no framework: plain DOM modules. Read `README.md` for how to run it, `CONTRIBUTING.md` for the development setup and an architecture overview, and `PRODUCT.md` for the design brief. This file covers how to change the code without making it harder to change next time.

## Before you finish any change

1. `cd web && npm run build`. This runs `tsc` and the Vite build. Both must pass with no new errors. After touching `main.go` or `internal/`, also run `go build ./...` and `go test ./...` from the repo root.
2. Look at what you changed. For anything visible, take a screenshot in headless Chromium in **both** light and dark themes (`Emulation.setEmulatedMedia` with `prefers-color-scheme`). You can import modules straight from the dev server to set up state, e.g. `await import('/src/items/diagram.ts')`.
3. Reload the page and check that your change survives restore from the saved layout.
4. Say plainly what you verified and what you didn't.

## Architecture: where things go

```
web/src/
  main.ts      boot, toolbar, keyboard shortcuts; imports features (importing a feature registers it)
  lib/         no knowledge of the app: api, store (persistence), blobs (IndexedDB), dom helpers, markdown, select, fonts, zoom (the figure zoom/pan dialog), connection (server reachability), tabs (which tab owns the stream and the saving), update (the update-available dialog), theme, tooltip, keys (the shortcut registry), help (the ? sheet and launch tips), sendkey (which key sends a message)
  canvas/      the canvas engine: view, items, window shape, graph edges, ink, references registry
  session/     session cards: card, composer, stream rendering, asks, live connection, history
  items/       one file per kind of canvas item: notes, sketch, diagram, plan, snippet, git, image, github (the window and its lists; + gh.ts, its data, Send to Claude and `publish()`; ghpr.ts, ghissue.ts, ghruns.ts: a pull request, an issue, Actions and checks), agent (a sub-agent's window), doc (a Markdown window), preview (a project file opened from Ctrl+K), group (a frame holding windows and drawings; + groupgeom.ts, its geometry, groupink.ts, the drawings it holds, and groupselect.ts, grouping from the selection)
  panels/      side panels: file tree + inspector, diffs
  styles/      index.css imports tokens.css, then one stylesheet per area
```

The dependency direction is `lib` ← `canvas` ← `items` / `session` / `panels` ← `main`.
- `lib/` never imports from other folders.
- `canvas/` never imports from `items/`. It only imports *types* from `session/` and `panels/`.
- If you need to import upward, add a registry or callback in the lower layer instead (see `onDrop`, `persist`, `referable`).

Import cycles between feature modules are tolerated only when every cross-use happens inside functions, never at module top level. Don't add top-level code that reads another module's exports.

## The registries: extend by adding, not by editing

The app scales through these registration points. A new feature should plug into them rather than add special cases elsewhere.

| Need | Use | Where |
|---|---|---|
| Something on the canvas | `addItem(el, kind)` for bare nodes, or `makeWindow({...})` for windows | `canvas/canvas.ts`, `canvas/window.ts` |
| Survive a reload | `persist(key, save, load, phase)` | `lib/store.ts` |
| Referenceable with `@` or by dropping on a card | `referable(kind, { icon, name, label, content })` (`name`: what Ctrl+K calls the kind) | `canvas/refs.ts` |
| Recover after the server comes back | `onReconnect(fn)` | `lib/connection.ts` |
| Post-process rendered Markdown (diagrams, anything drawn from a code block) | `onRendered(fn)` | `lib/markdown.ts` |
| Claude can create or edit it (canvas tools) | `creatable(kind, { size, create, update })` | `canvas/tools.ts` |
| Removed as part of a deleted selection, without its own confirm | `removable(kind, fn, note?)` (`fn` only if its × button asks first or it has none; `null` means its × is clicked; `note` words the selection's delete confirm) | `canvas/select.ts` |
| Off the page for now, may come back (a finished sub-agent's window, a delete Undo can still reverse) | `park(el)` returns what puts it back; `drop(el)` when it's gone for good; `onGone(fn)` for what holds items by id (a group's members), which must skip `parked(el)` removals | `canvas/canvas.ts` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HimalayanNomads/drawa](https://github.com/HimalayanNomads/drawa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
