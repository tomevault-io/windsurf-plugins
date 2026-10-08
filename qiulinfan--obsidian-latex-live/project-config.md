---
trigger: always_on
description: - Editor and gutter surfaces are transparent so vault/theme background-image snippets can
---

# obsidian-latex-live agent guidance

- Editor and gutter surfaces are transparent so vault/theme background-image snippets can
  paint behind them; keep text, selection, tooltip and control opacity independent. Background
  artwork and personal appearance settings belong to the vault's configuration, not this plugin.
  The preview's build controls and diagnostics live in `.ll-preview-dock`, in normal layout
  flow above the independently scrolling PDF viewport. Diagnostics start collapsed, never
  auto-open on errors, retain an explicit choice across builds, and reset on root changes or
  cleared diagnostics. Long lists scroll within the dock without covering the PDF.
  Editor font size, line height and source font overrides default to the host theme;
  MathJax preview scale defaults to 100%. `editorAppearance.ts` changes local CSS variables
  only. `TexView.applyAppearance` defers during IME and toggles a bounded pair of empty CM
  themes in its compartment to invalidate font metrics (requestMeasure alone misses short
  documents with a fixed min-height). Preserve document, selection, history and the original
  CHTML inline scale. Portaled tooltips copy the view's local typography; images, PDF crops,
  spinner/error text and PDF typography are independent. Run `editor-appearance-smoke.mjs`
  after changing this wiring, including short source documents and gutter alignment.
  With any nonempty selection, `texEditorExtensions` adds `ll-has-selection` through
  editorAttributes and the active-line background becomes transparent: drawSelection
  paints below content, so an opaque current line would hide its selected region.
  Keep the cursor-only active-line paint and selection-layer order unchanged. Check
  `selection-smoke.mjs` for forward/reverse/wrapped/dark pixel coverage and restoration.

- This plugin is a thin Obsidian frontend over the TeX distribution already
  installed on the machine (pdfLaTeX, XeLaTeX, LuaLaTeX, latexmk, SyncTeX).
  It owns the editor view, the compile queue, log parsing, the PDF preview,
  and Obsidian integration. Never bundle or reimplement a TeX engine.
- `src/tex/` must stay free of the `obsidian` module so it runs under Node
  tests against real TeX. Obsidian glue lives in `src/main.ts`,
  `src/session.ts`, `src/editor/`, and `src/preview/`.
- CodeMirror 6 and Lezer come from Obsidian at runtime and stay in the
  esbuild `external` list. Obsidian inlines its own CM6 language plumbing, so
  `syntaxHighlighting` never decorates custom views; highlighting goes through
  the decoration tokenizer in `src/editor/latexHighlight.ts`. The dev pins of
  `@codemirror/*` (`overrides` and devDependencies) are the versions the
  `minAppVersion` Obsidian ships (1.13.7: view 6.43.8, state 6.7.0), so tests run
  on the runtime's CodeMirror; raise them only together with `minAppVersion`.
- The PDF preview uses Obsidian's bundled pdf.js (`loadPdfJs`, 5.x in
  Obsidian 1.13). pdf.js takes ownership of the data buffer: always pass a
  copy (`slice()`). Every `getDocument` passes `PDFJS_ASSETS` (the cMap,
  standard font, wasm and ICC folders Obsidian's own viewer uses, under
  `/lib/pdfjs/`); without the CMaps the Chinese glyphs of XeLaTeX PDFs vanish.
- TeX tool conventions that bugs have already come from:
  - Pass the preamble format by name (`-fmt=<job>-preamble`) with
    `TEXFORMATS=<outDir>:`; an absolute `-fmt` path breaks the `-recorder`
    temp file name and pdfTeX exits before writing a log. A ready format has a
    stamp `<job>-preamble.json` (its key and the project files its preamble read,
    with mtimes), deleted before a rebuild and when the format proves broken;
    other runs (fragment compiles) take a format only through `readyPreambleFormat`.
  - Delete the job log before each run so a run that dies early never
    reports the previous log as success.
    A full latexmk no-op may reuse the existing PDF only when this invocation exits 0,
    has no new log, is neither stalled nor timed out, and explicitly confirms both this
    root's "Nothing to do" and its complete up-to-date PDF target list (XeLaTeX may also
    list XDV). Match logical/physical or root-relative paths exactly, including spaces;
    neither recent PDF mtime nor an old log proves success. The PDF must be readable.
    `pdfReused` distinguishes this result from a new compile; preview says "Up to date".
    Keep the real and fake regressions in `tests/fullBuildNoop.test.ts`.
  - TeX reports physical paths (`/private/var/...` on macOS); map them back
    with `logicalMapper` before comparing with vault paths.
    At the vault boundary, use Obsidian's `normalizePath` and NFC-normalized relative
    paths for its file index (including the vault root). Keep native filesystem/SyncTeX
    paths unchanged; do not URL-decode filenames. Reject outside paths after normalization too.
  - Spawn TeX with `max_print_line=10000` and `-file-line-error`; the log
    parser depends on both.
  - Every TeX run goes through `runTex` (`src/tex/run.ts`: its own process
    group, the stall watchdog, a timeout, an `AbortSignal` that kills the group;
    `Compiler.exec`, the export's probe and its dvisvgm run use it, `fragment.ts` still spawns with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qiulinfan/obsidian-latex-live](https://github.com/qiulinfan/obsidian-latex-live) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
