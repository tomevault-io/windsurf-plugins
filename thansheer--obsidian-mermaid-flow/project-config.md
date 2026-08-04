---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Mermaid Flow is an Obsidian community plugin: a visual WYSIWYG editor for Mermaid
flowcharts. TypeScript in `src/` is bundled by esbuild into a single `main.js`
that Obsidian loads. The repo lives *inside* a real vault at
`.obsidian/plugins/obsidian-mermaid-flow/`, so a `dev`/`build` writes `main.js`
into the very Obsidian instance that will run it — reload the plugin (or use
hot-reload) to test changes against this vault.

## Commands

```bash
npm run dev       # esbuild watch mode → rebuilds main.js on change (no typecheck)
npm run build     # tsc -noEmit typecheck + minified production bundle
npm run lint      # eslint over src/**/*.ts (relies on shell globbing the paths)
npm run validate  # assert manifest.json version === package.json version + required fields
npm version <v>   # runs version-bump.mjs → syncs manifest.json + versions.json, stages them
```

CI (`.github/workflows/ci.yml`) runs `validate` → `lint` → `test` → `build` on
push/PR to `main` — run all four locally before considering a change done.
Pushing a git **tag** triggers `release.yml`, which builds and attaches
`main.js`, `manifest.json`, `styles.css` to a GitHub release.

`@typescript-eslint/no-explicit-any` is off; `tsconfig.json` is otherwise
`strict` (incl. `noUncheckedIndexedAccess`), so expect to guard array/Map access.

## Architecture

### The round-trip core

The plugin is a visual wrapper around Mermaid text. Everything centers on one
in-memory shape, `DiagramModel` (`src/model.ts`), and converting to/from it:

```
Mermaid text ──parser.ts──▶ DiagramModel ──serializer.ts──▶ Mermaid text
                            (editor mutates
                             this in place)
```

- **`parser.ts`** — line-based, regex-driven, deliberately *forgiving*. It only
  understands the common flowchart subset. **Critical invariant:** any line it
  cannot interpret (classDef, click bindings, unknown directives, comments) is
  pushed verbatim into `model.extras` and re-emitted on save, so the visual
  editor never corrupts a user's advanced syntax. Preserve this when extending
  the parser — add real handling *or* let it fall through to `extras`, never drop.
- **`serializer.ts`** — `DiagramModel` → Mermaid, plus a `%% mermaid-flow:pos
  A=x,y[,w,h] …` comment that persists manual node positions (Mermaid ignores
  `%%` lines, so it stays valid). The parser reads this comment back. Gated by
  the `savePositions` setting / `includePositions` option.
- **`layout.ts`** — rank-based auto layout, used when parsed nodes have no saved
  positions (`layoutMissing`) or on explicit "Auto layout".

`model.ts` also holds all enum tables (`NodeShape`, `EdgeKind`, `Direction`) with
parallel `*_LABELS` maps, and model mutators (`removeNode`, `assignNodeToGroup`,
`cloneModel` for cancel/undo, id generators: nodes `A,B,…,N1,N2`; groups `sub1…`).

### Three edit entry points, one editor

`main.ts` (the `Plugin` subclass) wires three ways to start editing, which all
converge on `openEditor` → either a Modal or a pane:

1. **Editor commands / ribbon** — cursor-based. `editorBridge.ts` finds the
   `mermaid` block enclosing the cursor and writes back via the `Editor` API.
   Because the document can shift while the editor is open, `relocateBlock`
   re-scans a ±5-line window for the fence before replacing on save.
2. **Reading mode** — `registerMarkdownPostProcessor` adds an Edit/Code overlay.
   A `MutationObserver` re-attaches the overlay because Mermaid renders async and
   wipes it. Write-back uses `vault.process` (no live editor) against source
   lines from `ctx.getSectionInfo`.
3. **Live Preview** — `getSectionInfo` returns null for CM6 block widgets, so
   `editorExtension.ts` (a CM6 `ViewPlugin`) instead scans the doc for fences,
   watches the DOM for `.cm-embed-block` embeds, maps each back to a source line
   range via `posAtDOM`, and injects the same overlay. Line ranges come straight
   from editor state, so write-back is reliable.

Note: `OPEN_FENCE_RE` and the matching closing-fence builder (`closingFenceRe`)
live once in `diagramType.ts` and are imported by `main.ts`, `editorBridge.ts`,
and `editorExtension.ts` — change the shared definition, not a local copy.

### The visual editor (host-agnostic)

`DiagramEditorUI` (`editorUI.ts`, the largest file) is the whole editor —
toolbar, canvas, properties panel, raw-code view, undo/redo, autosave — and
renders into *any* container. It's hosted two ways behind the `EditorHost`
interface (`persist`/`close`/`autoSave`/`closeOnSave`):

- `editorModal.ts` — popup (`Modal`)
- `editorView.ts` — embedded workspace pane (`ItemView`, `VIEW_TYPE_MERMAID_FLOW`)

The `openMode` setting picks which. Autosave (debounced `persist`) applies only
to the embedded pane editing an existing block.

`canvas.ts` (`DiagramCanvas`) is the SVG interaction surface: drag, connect,
rubber-band multi-select, resize, subgraph drag. It **mutates the model in place**
and reports via `CanvasCallbacks` (`onSelect`/`onChange`/`onContextMenu`).
**Convention:** `node.x`/`node.y` are the node *centre*, not top-left.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [THANSHEER/obsidian-mermaid-flow](https://github.com/THANSHEER/obsidian-mermaid-flow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
