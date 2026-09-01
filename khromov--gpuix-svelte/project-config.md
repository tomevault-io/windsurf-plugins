---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Svelte custom renderer that targets GPUI (Zed's GPU-accelerated UI framework) through
`@gpuix/native`. Svelte components render into a real native desktop window — no DOM, no
webview, no browser. Built on Svelte's unreleased custom renderer API
([sveltejs/svelte#18511](https://github.com/sveltejs/svelte/pull/18511)).

## Commands

Everything goes through the package scripts. They embed
`node --conditions custom-renderer --conditions development --import ./src/register.js`. The
conditions are **mandatory** — without them `svelte` resolves to its server build and `mount()`
does not exist — and `--import` installs the `.svelte` loader.

```bash
npm install                # entire setup; @gpuix/native ships prebuilt, no Rust toolchain
npm run demo               # all four demos at once (via scripts/demo-all.js —
                           # cmd.exe has no `&` ... `wait`)
npm run demo:counter       # counter (hot-reloads on save)
npm run demo:tictactoe
npm run demo:hn            # Hacker News reader — live network data, scrolling
npm run demo:glass         # transparent window, macOS vibrancy (GPUI window blur)
npm run demo:glass-ffi     # same app on real Liquid Glass (NSGlassEffectView, macOS 26+)
                           # via a clang-compiled ObjC shim + FFI (koffi/bun:ffi) in
                           # examples/liquid-glass-ffi/glass.js; GPUIX_GLASS=0 forces the
                           # window-blur fallback; NOT part of `npm run demo`

npm test                   # test:reorder, test:smoke, test:autocommit, test:style,
                           # test:teardown, test:lifecycle, test:compile
npm run test:reorder       # single test — keyed {#each} reordering
npm run test:smoke         # single test — mount + click Counter headlessly
npm run test:autocommit    # single test — the microtask drain used where there is no frame loop
npm run test:style         # single test — CSS shorthand expansion, and what must never reach serde
npm run test:teardown      # single test — removal marks dirty, blank text demotes, listeners survive
npm run test:lifecycle     # single test — throws don't kill the frame loop; remount is one batch
npm run test:compile       # single test — the ?v=N cache-buster reaches every child specifier
npm run test:coverage      # optional; needs SVELTE_SAMPLES_DIR (see below)
```

Every command has a `bun:`-prefixed twin (`npm run bun:test`, `npm run bun:demo:counter`, ...)
running the same entry point through Bun, which takes the loader from `bunfig.toml` rather than
`--import`. Deps come from `npm install` either way. Adding a script means adding both halves.

To verify interactions, prefer `TestGpuixRenderer.simulateClick/simulateMouseDown/...` — they run
GPUI's real hit testing (occlusion included) and queue results for `drainEvents()`, which you feed
through `dispatch()`. Calling `dispatch()` directly injects events at an element and *bypasses* hit
testing, so it can pass while the real window fails. The headless viewport width follows
`new TestGpuixRenderer(width, height)`, but its height caps at 538 logical px — elements laid
out below that can't be hit (shift the layout up inside an absolute wrapper to reach them).

Tests are plain scripts that assert and `process.exit(1)` — no test runner.
Adding one means adding a `test:*` script and chaining it into `test`. CI (`.github/workflows/test.yml`)
runs `npm test` and `npm run bun:test` as two macOS jobs.

`test:coverage` mounts every sample from Svelte's own custom-renderer suite; point
`SVELTE_SAMPLES_DIR` at a svelte checkout's `packages/svelte/tests/custom-renderers/samples`
(it skips silently otherwise). It copies them into `test/.samples-tmp` first, because importing
in place would mix two Svelte runtimes.

### Seeing what a demo renders

A window can't be inspected from a terminal, but a PNG can:

```bash
GPUIX_SCREENSHOT=/tmp/x.png npm run demo:counter    # writes a PNG after every mount/remount
                                                    # (single demo — all four share the path)
```

Then open the PNG with the Read tool (Preview.app also reloads on write). Headless code calls
`TestGpuixRenderer.captureScreenshot(path)` — real Metal pipeline, no window; see `test/smoke.js`.

## Hard constraints

- **No build step, no TypeScript emit.** Plain ESM JS with JSDoc types; `exports` points straight
  at `src/*.js`. Keep it that way.
- **Node >= 24** (for `module.registerHooks`) or **Bun >= 1.4.0**. Both are tested in CI; keep
  runtime-specific code confined to `register.js` / `plugin.js`.
- **Never `bun --hot`.** `render_hot` implements its own in-process reload; `--hot` re-evaluates
  Svelte's runtime, so the old component belongs to a module instance the new one can't see and
  `unmount()` fails.
- **`svelte` is pinned to `https://pkg.pr.new/svelte@18511`** (CI preview of the custom-renderer
  branch). The committed `package-lock.json` pins the exact version and integrity hash, but its
  `resolved` is that same URL — pkg.pr.new has to be live for `npm ci` too, not just
  `npm update svelte`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [khromov/gpuix-svelte](https://github.com/khromov/gpuix-svelte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
