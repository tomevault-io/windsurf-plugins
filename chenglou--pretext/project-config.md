---
trigger: always_on
description: Use `README.md` as the public source of truth for API examples and user-facing limitations. See `DEVELOPMENT.md` for the current command surface and the canonical dashboards/snapshots to consult before making browser-accuracy or benchmark claims. Use `TODO.md` for the current priorities. **Every time before you commit, ensure you've synced the docs**.
---

## Pretext

Use `README.md` as the public source of truth for API examples and user-facing limitations. See `DEVELOPMENT.md` for the current command surface and the canonical dashboards/snapshots to consult before making browser-accuracy or benchmark claims. Use `TODO.md` for the current priorities. **Every time before you commit, ensure you've synced the docs**.
Do not change the existing tone of the documents unless they're wrong.
Do `bun install` if you're in a fresh worktree.

**Important:** after you're done with a feature, and have enough holistic vision, make sure you do a pass over all the files again and see if you can simplify anything. Don't change things for the sake of, but if there are simplifications, YELL **I DID A HOLISTIC PASS AND FOUND SIMPLIFICATIONS** with a brief summary.

**Important:** do NOT monkey-patch. If you found yourself solving the symptom instead of the root cause, reconsider and do a proper fix, then YELL **I SOLVED THE ROOT CAUSE NOT THE SYMPTOM** with a brief summary.

Changelog updates guideline: don't add dev-facing notes, only user-facing ones. Refer to closed PR numbers.

### Commands

See `DEVELOPMENT.md` for the current command surface and packaging/release checks. Keep the higher-level workflow notes below in sync with that command list rather than duplicating it here.

### Important files

- `package.json` — published entrypoints now target `dist/layout.js` + `dist/layout.d.ts`; keep the package/export surface aligned with the emitted files
- `tsconfig.build.json` — publish-time emit config for `dist/`
- `scripts/package-smoke-test.ts` — tarball-level JS/TS consumer verification for the published package shape
- `src/layout.ts` — core library; keep `layout()` fast and allocation-light
- `src/analysis.ts` — normalization, segmentation, glue rules, and text-analysis phase for `prepare()`
- `src/measurement.ts` — canvas measurement runtime, segment metrics cache, emoji correction, and engine-profile shims
- `src/line-break.ts` — internal line-walking core shared by the rich layout APIs and the hot-path line counter
- `src/bidi.ts` — simplified bidi metadata helper for the rich `prepareWithSegments()` path
- `src/rich-inline.ts` — inline-only helper for rich-text inline flow, atomic pills, and boundary whitespace collapse
- `src/test-data.ts` — shared corpus for browser accuracy pages/checkers and benchmarks
- `src/layout.test.ts` — small durable invariant tests for the exported prepare/layout APIs
- `pages/accuracy.ts` — browser sweep plus per-line diagnostics
- `status/dashboard.json` — machine-readable main status dashboard derived from the checked-in accuracy and benchmark snapshots
- `accuracy/chrome.json` / `accuracy/safari.json` / `accuracy/firefox.json` — checked-in raw accuracy rows
- `pages/benchmark.ts` — performance comparisons
- `benchmarks/chrome.json` / `benchmarks/safari.json` — checked-in current benchmark snapshots
- `corpora/dashboard.json` — machine-readable long-form corpus dashboard derived from the corpus snapshots and notes
- `corpora/chrome-step10.json` / `corpora/safari-step10.json` — checked-in browser `step=10` corpus sweep snapshots
- `pages/diagnostic-utils.ts` — shared grapheme-safe diagnostic helpers used by the browser check pages
- `scripts/pre-wrap-check.ts` — small permanent browser-oracle sweep for the non-default `{ whiteSpace: 'pre-wrap' }` mode
- `pages/demos/index.html` — public static demo landing page used as the GitHub Pages site root
- `pages/demos/bubbles.ts` — bubble shrinkwrap demo using the rich non-materializing line-range walker
- `pages/demos/dynamic-layout.ts` — fixed-height editorial spread with a continuous two-column flow, obstacle-aware title routing, and live logo-driven reflow
- `pages/demos/markdown-chat.ts` — rich chat virtualization demo that stress-tests prepared templates and manual block layout
- `pages/demos/rich-note.ts` — inline-rich-note demo that dogfoods the rich-text inline flow helper at `@chenglou/pretext/rich-inline`

### Implementation notes

- The published package ships built ESM from `dist/`; `dist/` is publish-time output, not checked-in source.
- Keep shipped library source imports runtime-honest with `.js` specifiers inside `.ts` files. That keeps plain `tsc` emit producing correct JS and `.d.ts` files without a declaration rewrite step.
- `prepare()` / `prepareWithSegments()` do horizontal-only work. `layout()` / `layoutWithLines()` take explicit `lineHeight`.
- `setLocale(locale?)` retargets the hoisted word segmenter for future `prepare()` calls and clears shared caches. Use it before preparing new text when the app wants a specific `Intl.Segmenter` locale instead of the runtime default.
- `prepare()` should stay the opaque fast-path handle. If a page/script needs segment arrays, that should usually flow through `prepareWithSegments()` instead of re-exposing internals on the main prepared type.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chenglou/pretext](https://github.com/chenglou/pretext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
