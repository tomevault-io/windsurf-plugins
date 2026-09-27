---
trigger: always_on
description: This file applies to the entire repository.
---

# AGENTS.md

This file applies to the entire repository.

## Project overview

Mktero is a restartless Zotero extension for Zotero 7 through 10. It reads a
local PDF attachment, sends it to the selected OCR provider (MinerU or Mistral
OCR) when needed, and opens the resulting Markdown, figures, citations,
annotations, and optional AI translation in a session-only, reading-first
Zotero tab.

The project is plain JavaScript, not TypeScript. Source files use ES modules,
while esbuild bundles the runtime entry points as browser IIFEs for Firefox 115
(the bootstrap, the preferences UI, and the figure render worker), plus the
bundled pdf.js worker. Zotero supplies the privileged runtime globals used by
the extension, including `Zotero`, `IOUtils`, `PathUtils`, `Components`,
`ChromeUtils`, and `Services`.

## Commands

Use the Node.js version in `.node-version` (currently 24.15.0); it is the
shared local and CI version source. Do not use Node.js 25, which is outside the
supported dependency engine range.

```bash
npm ci
npm run check
npm test
npm run build
```

- Run one test file with `node --test test/<name>.test.js` while iterating.
- `npm run check` syntax-checks every source module explicitly. Add new source
  files to the `check` script in `package.json`.
- `npm run build` recreates `build/package`, the reproducible
  `build/mktero-<version>.xpi`, its `.sha256` checksum, and `build/updates.json`.
  Both `build/` and `node_modules/` are generated and ignored; never edit or
  commit them.
- `scripts/build.mjs` has an explicit list of copied runtime assets. Update it
  when adding a non-imported file required by the packaged extension.

Before handing off a change, run the narrow tests for the touched behavior,
then `npm run check`, `npm test`, and `npm run build` unless the change is
documentation-only. Report any command that could not be run.

## Repository map

- `manifest.json`: Zotero compatibility, extension ID, version, and update URL.
- `prefs.js`: defaults for every Zotero preference.
- `src/bootstrap.js`: extension lifecycle and dependency composition. It owns
  startup/shutdown, conversion cancellation, tabs, toolbar actions, context
  menus, preferences, and cache.
- `src/ai/`: Vercel AI SDK gateway, translation service, and request tracking.
- `src/cache/`: content-addressed Markdown, figure, translation, citation-graph,
  and reading-position stores under the active Zotero profile.
- `src/citations/`: citation-graph building and the Semantic Scholar,
  OpenCitations, and OpenAlex clients.
- `src/config/`: preference keys, preference-pane registration, and the
  session-only models.dev reasoning catalog used by the AI reasoning menu.
- `src/core/`: provider-independent conversion orchestration and progress.
- `src/extractors/`: adapters from Zotero items or provider results to the core
  document shape.
- `src/figures/`: figure region resolution, panel/label recovery, reading-order
  reunion, and the worker-based restoration pipeline.
- `src/i18n/`: English and Simplified Chinese user-facing copy.
- `src/icons/`: bundled lucide icon helpers.
- `src/mineru/`: MinerU API client, parsing profile, ZIP extraction, binary
  helpers, and Markdown normalization.
- `src/mistral/`: Mistral OCR client, parsing profile, and result normalization.
- `src/pdf/`: PDF.js text engine, page crops, annotation location, and outlines.
- `src/platform/`: Zotero/runtime adapters for aborting requests and Zotero APIs.
- `src/markdown/`: pure Markdown parsing, analysis, normalization, rendering,
  and safety logic.
- `src/editor/`: CodeMirror 6 presentation, inline correction editing, image
  previews, and citation/table/figure interactions.
- `src/ui/`: Zotero toolbar, item menu, tab presenter/view, loading state, and
  preference controller.
- `ui/`: packaged XHTML, CSS, and icons.
- `docs/`: README-linked user documentation. The directory is gitignored except
  for an explicit allowlist in `.gitignore`; see the cross-file checklist.
- `scripts/`: build, figure validation harnesses, and the figure-corpus check.
- `test/`: Node test runner coverage, with jsdom or linkedom where DOM behavior
  is needed.

## Architecture and runtime rules

- Keep `src/bootstrap.js` as the composition root. Prefer constructor or
  function injection over importing Zotero globals into pure modules.
- Preserve the dependency direction: UI and runtime adapters may depend on
  core and pure helpers; core and pure helpers must not depend on Zotero UI.
- Use explicit `.js` extensions in imports. Do not introduce Node-only APIs
  into files bundled for Zotero.
- Preserve all bootstrap lifecycle globals: `install`, `startup`, `shutdown`,
  `uninstall`, `onMainWindowLoad`, and `onMainWindowUnload`.
- Every registration, listener, object URL, tab, and in-flight
  conversion needs a matching cleanup path. Closing a tab or shutting down the
  extension must abort its active conversion.
- Mktero tabs are session-only and there is at most one live tab per PDF item.
  Do not make them restorable without redesigning stale-tab cleanup and tests.
- The Markdown reading surface is intentionally read-only. Keep the document
  `EditorView.editable.of(false)` / `EditorState.readOnly.of(true)`. The only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tenglvjun/mktero](https://github.com/tenglvjun/mktero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
