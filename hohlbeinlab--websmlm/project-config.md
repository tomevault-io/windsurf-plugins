---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This file describes the **current** state of the codebase and standing conventions — not a
chronological history of how it got there. Shipped-feature history (specific bug reports, rejected
approaches, exact before/after numbers) lives in [`CHANGELOG.md`](CHANGELOG.md), which is the place
to check "why did we do X" for anything not covered below; forward-looking ideas live in
[`docs/REFACTOR_PLAN.md`](docs/REFACTOR_PLAN.md). Keep this file that way: when you fix something,
update the relevant paragraph below to reflect the new *current* behavior rather than appending a new
"reported... fixed..." entry — the report itself belongs in the commit message and CHANGELOG.md.

## What this is

webSMLM is a **single-file** browser tool for single-molecule localization microscopy (SMLM):
the entire application — HTML, CSS, all JavaScript, and the two bundled decoders (pako, UTIF) —
lives in `webSMLM.html` (~20,800 lines; the file's own top-of-file **MODULE INDEX** comment gives
current per-module line numbers — re-`grep -n "MODULE:"` if it looks stale, and refresh it alongside
a build-letter bump when a change has moved things by more than a few lines). It loads a raw TIFF
stack, detects/localizes emitters, and renders a super-resolution image, **entirely client-side** (no
upload, no server, no network calls at runtime). `index.html` is just a redirect to `webSMLM.html`
for the bare Pages URL.

`webSMLM.html` itself has **no build system, no package.json, no dependency install, and no test
runner.** "Running" the app = opening `webSMLM.html` in a browser (double-click, or the hosted
Pages copy). Do not introduce a bundler, framework, or npm dependency to the app itself — the
zero-install single-file property is the point. New third-party code must be inlined and its
license honoured in the head banner. (`tools/` is the one exception: a separate, optional
Node+Playwright CLI for headless/scripting use — see **pipeline** below — with its own scoped
`package.json`, deliberately kept out of `webSMLM.html` so the app's own property is untouched.)

## Editing model

All work happens inside `webSMLM.html`. It is organized into commented `MODULE:` banners; find the
relevant one before editing rather than scrolling. The code itself carries extensive inline "why"
comments at nearly every non-obvious decision — the summaries below are a map to get oriented and a
place to record cross-cutting facts, not a substitute for reading the code's own comments once you're
in a module.

- **params** — the `PARAMS` registry: single source of truth for every analysis/render/export
  parameter (`name → {label, min, max, step, default, int}`), read via `paramValue(id)`. Drives the
  HTML controls' min/max/default (`syncParamControls()`), Save/Load Settings, and the headless
  `window.webSMLM.analyze(config)` config — a new `PARAMS` entry is automatically available to both
  with no extra wiring. Deliberately excludes pure display/layout (CSS) and per-dataset working state
  (`calFirst`/`calLast`/`zmin`/`zmax`).

  `addNumberSteppers()` wraps every `input.num` in a `.numstep` span with an always-visible
  Inkscape-style `.numstep-btns` −/+ pair (not the browser's native spinner — inconsistent look
  across engines, hover-reveal only, unreachable on touch). Reads each input's already-present
  `min`/`max`/`step`, so any current or future `.num` field gets steppers for free. Clicking
  dispatches real `input`/`change` events.

  `pxnm` ("Pixel size (nm)") and `frametime` ("Frame time (s)") are pinned always-visible near the
  top of the sidebar, outside any collapsible section — both are per-dataset acquisition properties
  several modules (spt, smFRET) depend on, not settings local to one module. `gain`/`camoffset`/**Get
  estimate** sit at the top of **Localisation**, right below **Real-time update**.

  **No checkbox or control label ends in "?"** — a plain house-style convention (e.g. **Apply
  segmentation**, **3D localisation**, **Analyse FRET**, **Position donor**). `select.sel:disabled`
  needs its own explicit `{opacity:.45;cursor:not-allowed}` rule — its `color:var(--fg)` defeats a
  browser's native disabled-dimming, so `.disabled=true` alone is invisible without it.

- **in/out** — TIFF/ND2/FITS parsing; in-memory vs. streamed loading; handles multi-GB files via
  `File.slice()` (never fully loaded). `loadTiffFile()`'s dispatch chain: FITS (`isFitsFile()`, magic
  byte) → ND2 (`isNd2File()`, magic `0x0ABECEDA`) → TIFF-in-disguise (`t256`/`t257` sanity-checked —
  UTIF returns one EMPTY ifd object, no exception, on non-TIFF bytes) → whole-file
  (`file.arrayBuffer()`) vs. streamed (`loadMultiIfdStreaming()`), gated by
  `effSliceMin=min(SLICE_MIN≈1.5GB, readBudget())` — ties the streaming threshold to **Budget raw
  movies (GB)** (`memgb`), which defaults to `0` (not `3`) on a memory-constrained device
  (`isMemoryConstrainedDevice()`, `MOBILE_MEM_DEFAULTS`/`syncParamControls()`, MODULE: params) — a
  `0` budget floors `effSliceMin` at `0`, so EVERY movie load on such a device takes the streamed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HohlbeinLab/webSMLM](https://github.com/HohlbeinLab/webSMLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
