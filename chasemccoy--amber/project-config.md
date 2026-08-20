---
trigger: always_on
description: Guidance for working in the **amber** codebase.
---

# CLAUDE.md

Guidance for working in the **amber** codebase.

## What this is

amber turns a URL into a self-contained, de-junked offline folder: an
`index.html` + local `assets/` tree with every reference rewritten to a local
path, the junk (ads, cookie banners, trackers) removed, and embedded media
downloaded as a real file. The design splits the work in two: deterministic
Node does the mechanical capture/rewrite/package; `claude-sonnet-4-6` does the
judgement (what's junk, what's the main content, which embeds are real media).

## Commands

```bash
pnpm install
pnpm exec playwright install chromium   # one-time, for the render backend

pnpm archive <url>     # one-shot pipeline (src/cli.ts) — capture, plan, clean, package
pnpm agent <url>       # local agentic loop (agent/agent.ts) — Claude drives the tools

pnpm test              # deterministic unit tests — no key, no network, no browser
pnpm typecheck         # tsc --noEmit
pnpm evals             # vitest-evals judgement suite (LLM/agent suites need a key)
pnpm build             # tsup → dist/ (only needed for publishing; dev runs tsx directly)
```

Always run `pnpm typecheck` and `pnpm test` after changing `src/`. After
touching the planner/agent/judges, run `pnpm evals` (it makes real API calls and
needs `ANTHROPIC_API_KEY`).

## Architecture

Two entry points share the same capture/clean code:

- **Pipeline** (`pnpm archive`) — one structured-output call returns a plan that
  the pipeline applies. Stages in `src/pipeline.ts` `archiveUrl()`/`finishArchive()`:
  1. **Capture** — `src/render.ts` (Playwright) + `src/capture.ts`. Backend is
     `auto` by default: static `fetch` first, escalating to a headless-Chromium
     render only when the page looks client-rendered (`assessRendering`).
  2. **Plan** — `src/planner.ts`. `llmPlan()` judges the *raw pre-capture* HTML
     (`messages.parse` + a Zod schema); `heuristicPlan()` is the no-key fallback.
     The plan includes `preserveRuntime` — Claude's judgement of whether the
     page's presentation needs its JS at view time. Plan resolution happens in
     `archiveUrl()` (via `resolvePlan()`), *before* clean/localise, so a true
     verdict can re-capture the page in keep-js mode (`--keep-js` forces it,
     `--no-keep-js` forbids it; auto-escalation needs esbuild + Playwright and
     is skipped under `--static`). Only the LLM plan sets it; heuristics never
     escalate. Guarded by `evals/runtime.eval.ts` (fixtures in `shared.ts`
     `RUNTIME_FIXTURES` — the false cases are deliberately tempting).
  3. **Clean & localise** — junk is removed *before* assets are downloaded, so
     junk-only bytes are never fetched: `removeJunk()` (protecting the plan's
     media embeds via `mediaTargets()`) + `stripStatic()` (unconditional:
     `<script>`/`<noscript>`/preconnect & prefetch hints, `on*` handlers,
     `javascript:` hrefs, comments), then `captureAssets()` walks the surviving
     DOM, downloads every referenced asset (incl. `url()` in `<style>` blocks
     and `.css` files), and rewrites refs to **relative local paths**
     (`<a href>`/`<iframe src>` left alone; `<script src>` never downloaded),
     then `swapMedia()` downloads embeds via yt-dlp and swaps them in.
     `applyPlan()` in `src/clean.ts` still composes swap→junk→strip in one call
     for tests/evals that clean an already-captured DOM.
  4. **Package** — writes `index.html`, `plan.json`, `manifest.json`, and a
     `thumbnail.jpg` (live-render viewport when a browser ran, else a
     screenshot of the staged archive via `thumbnailFromFile()`; excluded from
     the content hash so dedupe still works). After commit, both entry points
     (and agent mode) rebuild the **library index** — `src/library.ts` scans
     the slug folders' manifests and writes a browsable `index.html` at the
     archive root (`amber index` rebuilds it manually). Pure projection: the
     folders are the database, no state file. Tagging is filing-oriented
     (discipline-altitude, spaces not hyphens — rules live in the planner
     prompt) and converges: `resolvePlan` feeds `libraryTags(outRoot)` into
     every planning call so the model reuses the library's vocabulary.

  **Keep-js mode** (`src/keepjs.ts`) — preserves a page's own runtime when the
  experience *is* the JS (WebGL, scroll choreography). During the Playwright
  render (with `deterministicRandom` — Math.random seeded identically in render
  and replay — plus a denser auto-scroll), every response is recorded; then:
  trackers removed by heuristic, module scripts flattened to one classic IIFE
  via esbuild (recorded responses as the virtual fs, network fallback for lazy
  chunks), a replay shim injected (patches fetch/XHR from an embedded response
  map, remaps runtime-constructed element src/poster/href through an asset map
  with nearest-variant fallback, stubs beacons/WebSocket), runtime-only assets
  localised with numeric-sequence gap-filling, and finally
  `finalizeKeepJsDelivery()` collapses everything into ONE self-contained
  `index.html` with assets as data: URIs — that's what lets canvas/WebGL run
  from a double-clicked file:// page (file:// taints local media otherwise).
  The inliner STREAMS the output (DOM holds `@@AMBER[rel]@@` tokens, expanded

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chasemccoy/amber](https://github.com/chasemccoy/amber) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
