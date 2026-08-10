---
trigger: always_on
description: Guidance for working on this repo. The plugin is **shipped** (GitHub releases cut,
---

# CLAUDE.md

Guidance for working on this repo. The plugin is **shipped** (GitHub releases cut,
in/through Obsidian community review). Most future work is adding features or
fixing bugs on a stable base — so the priority is **not regressing the non-obvious
decisions below**, many of which were made to satisfy the plugin-review scanner or
to work around drawio/Obsidian quirks.

## What this is

An Obsidian plugin that embeds, previews, and edits
[draw.io](https://www.drawio.com/) (diagrams.net) diagrams. Plugin **id is
`drawio-editor`** (the bare `drawio` id is reserved — do not change it back).
**Editing is desktop-only** (it needs the iframe-based drawio embed app, the
local Node server, or a network connection); **mobile (phone/tablet) gets
preview only** — code blocks, embeds, and a read-only view for standalone
`.drawio` files. See `src/desktop/registerDesktopFeatures.ts` and the
"Mobile support" entry below.

Three surfaces:
- **Code blocks** — ` ```drawio ` blocks: rendered as an SVG preview, click to edit.
- **Standalone `.drawio` files** — opened in a dedicated tab with the editor embedded inline (Excalidraw-style).
- **Embeds** — `![[file.drawio]]` in any note: inline preview in both editing and reading views, click to edit.

## Two independent rendering engines (important mental model)

1. **Editing** uses the drawio **embed app** in an `<iframe>` over the `postMessage`
   JSON protocol (`src/editor/`). Source = online `embed.diagrams.net`, the bundled
   offline webapp via a local server, or a custom URL.
2. **Previews** use the bundled **`viewer.min.js`** (drawio's `GraphViewer`) to
   produce a static, sanitized SVG (`src/preview/`). Fully offline, **no iframe, no
   network** — the viewer is bundled into `main.js`.

These are separate; a change to one rarely affects the other.

## Module map

- `src/main.ts` — plugin entry: settings, local server, registers the code-block
  processor / file view / embeds / `Create new diagram` command / settings tab.
  `resolveBaseUrl()` picks the editor URL (offline → local server; throws a typed
  `OfflineEditorNotInstalledError` when the webapp isn't installed — the automatic
  online fallback was removed after 0.4.x, **no automatic fallback** anymore).
- `src/constants.ts` — view type, file ext, `ONLINE_DRAWIO_URL`, `EMPTY_DIAGRAM`, `buildEmbedQuery`.
- `src/settings.ts` / `src/settingsTab.ts` — settings model + settings tab.
- `src/model/` — `DrawioSource` (edit-target abstraction: code block or file),
  `xmlUtils` (`isValidDrawioXml`/`ensureMxfile`), `formatXml` (pretty-print),
  `codeBlockEdit`/`locateBlock` (find & replace a block's XML in a note).
- `src/codeblock/` — code-block processor + `CodeBlockSource`.
- `src/file/` — `DrawioFileView` (inline-editor tab, a `TextFileView`),
  `EmbedRenderer` (via `app.embedRegistry`, with a Reading-view post-processor
  fallback), `FileSource`.
- `src/editor/` — `DrawioEditor` (iframe + postMessage), `DrawioModal`, `embedMessages`.
- `src/preview/` — `ViewerRenderer` (`renderPreview`), `loadViewer`, `svgSanitizer`,
  `editHint`, `pageControl` (multi-page prev/next control), and the vendored
  `viewer.min.txt`.
- `src/server/` — `ServerManager` (local `127.0.0.1` HTTP server serving the offline
  webapp, with idle shutdown) + `portDetector`.

## Build / test / dev

- `npm run fetch-drawio` — **run once before building.** Downloads pinned drawio
  (`draw.war`, v30.0.4) into `webapp/` and copies `js/viewer.min.js` →
  `src/preview/viewer.min.txt`. Both `webapp/` and `viewer.min.txt` are **gitignored**
  (so a fresh clone must run this first). Needs network + `unzip` or `python3`.
- `npm run build` — `tsc -noEmit` then esbuild production bundle → `main.js` (gitignored).
- `npm run dev` — esbuild watch.
- `npm test` — vitest (unit tests in `tests/`).

Local manual testing installs to a vault by copying `main.js` + `manifest.json` +
`styles.css` (and optionally `webapp/`) into `<vault>/.obsidian/plugins/<folder>/`.
The plugin folder used during development is named `obsidian-drawio` (the old
name; the manifest id inside is `drawio-editor`).

## Non-obvious decisions — DO NOT casually revert

- **Mobile support (`isDesktopOnly: false`)**: `main.ts` and `ServerManager.ts`
  used to have top-level static imports of `node:http`/`node:fs`/`node:path`.
  esbuild marks Node built-ins as `external` (`esbuild.config.mjs`), so a
  static top-level import compiles to an unconditional, module-load-time
  `require(...)` call — which throws immediately on mobile (no Node runtime),
  crashing the *entire plugin load* before `onload()` even runs. Fixed by
  never letting a `node:*`/`electron` import be *static and top-level* outside
  `src/server/**` or `src/desktop/**` — both are only ever reached through
  `Platform.isDesktopApp`-gated dynamic imports: `main.ts`'s
  `maybeRegisterDesktopFeatures()`, and `settingsTab.ts`'s `startInstall()`
  (whose offline-editor row only renders inside the tab's desktop-gated
  block). A dynamic `await import(...)` at the point
  of use (e.g. `main.ts`'s `pluginDir()`/`resolveBaseUrl()`) is fine anywhere,
  since it's never eagerly evaluated — only a *static* top-level import gets
  hoisted and unconditionally `require()`d. **If you add a new Node/Electron

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doge-liang/obsidian-drawio](https://github.com/doge-liang/obsidian-drawio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
