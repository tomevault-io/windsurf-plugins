---
trigger: always_on
description: Drawa is a browser canvas around the Claude Code CLI. `main.go` + `internal/` run `claude` processes and serve a JSON API. `web/` is a Vite + TypeScript frontend with no framework: plain DOM modules. Read `README.md` for how to run it, `CONTRIBUTING.md` for the code layout, and `PRODUCT.md` for the design brief. This file covers how to change the code without making it harder to change next time.
---

# Guidelines for agents working on Drawa

Drawa is a browser canvas around the Claude Code CLI. `main.go` + `internal/` run `claude` processes and serve a JSON API. `web/` is a Vite + TypeScript frontend with no framework: plain DOM modules. Read `README.md` for how to run it, `CONTRIBUTING.md` for the code layout, and `PRODUCT.md` for the design brief. This file covers how to change the code without making it harder to change next time.

## Before you finish any change

1. `cd web && npm run build`. This runs `tsc` and the Vite build. Both must pass with no new errors. After touching `main.go` or `internal/`, also run `go build ./...` and `go test ./...` from the repo root.
2. Look at what you changed. For anything visible, take a screenshot in headless Chromium in **both** light and dark themes (`Emulation.setEmulatedMedia` with `prefers-color-scheme`). You can import modules straight from the dev server to set up state, e.g. `await import('/src/items/diagram.ts')`.
3. Reload the page and check that your change survives restore from the saved layout.
4. Say plainly what you verified and what you didn't.

## Architecture: where things go

```
web/src/
  main.ts      boot, toolbar, keyboard shortcuts; imports features (importing a feature registers it)
  lib/         no knowledge of the app: api, store (persistence), blobs (IndexedDB), dom helpers, markdown, select, fonts, zoom (the figure zoom/pan dialog), connection (server reachability), theme, tooltip
  canvas/      the canvas engine: view, items, window shape, graph edges, ink, references registry
  session/     session cards: card, composer, stream rendering, asks, live connection, history
  items/       one file per kind of canvas item: notes, sketch, diagram, plan, snippet, git, image, github (+ gh.ts, its data and Send to Claude), agent (a sub-agent's window), doc (a Markdown window)
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

**Adding a new kind of canvas item** should mean one new file in `items/`, an import in `main.ts`, and CSS in `styles/items.css`. The item file should:
- Build the element with `makeWindow()`, which handles the folder tab, dragging, collapsing and resizing.
- Call `persist()` for its saved state, and restore a list with `each(list, fn)` from `lib/store.ts` so one bad entry doesn't stop the rest. Binary data (images) goes in IndexedDB via `lib/blobs.ts`, keyed by the item's id; localStorage only holds the layout.
- If its content scales with the window (a picture, a drawing), put it in `inkBox()` from `canvas/ink.ts`, so pen strokes on it keep their spot at any size, including full view.
- Something Claude should receive that isn't on the canvas (a GitHub pull request) can still be a reference: `referable()` a kind, then `addRef()` a detached element whose dataset says what to fetch at send time (see `sendToClaude` in `items/gh.ts`). Such chips have no arrow; clicking one runs the element's `onclick`.
- Text marked in place (like pinned snippets' sources in `items/pinmarks.ts`) uses the CSS Custom Highlight API, never wrapper elements: the chat re-renders while streaming and skips off-screen rows.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [probablysamir/drawa](https://github.com/probablysamir/drawa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
