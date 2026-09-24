---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**Note by Note** (package name `note-by-note`) is a Chrome/Firefox MV3 extension for practicing music along with any browser audio/video: pitch shift, speed, loop ranges, timeline markers, chained practice snippets, vocal reducer, and 10-band EQ. Built with WXT + Svelte 5 (runes) + TypeScript. Dev environment is Windows/PowerShell — chain shell commands with `;`, use `pnpm`.

## Commands

```powershell
pnpm install          # runs postinstall: wxt prepare + generates both worklet bundles
pnpm dev              # launch Chrome with the extension + HMR (uses a persistent .wxt/chrome-data profile)
pnpm dev:firefox      # same, Firefox
pnpm check            # svelte-check / TypeScript — the only type/lint gate
pnpm build            # production build → .output/chrome-mv3
pnpm zip              # store package
pnpm test:dsp         # fast unit tests: DSP, chords, library and sync records (node --test on src/**/*.test.ts)
pnpm release:dry      # show the release plan (version bump, tag) without changing anything
pnpm release          # full release: check + test, bump patch, build both zips, commit, tag, push
```

[scripts/release.ps1](scripts/release.ps1) is the release path. It refuses to run on a dirty tree, on a
branch other than `main`, when `main` is behind `origin/main`, or when the tag already exists; unpushed
commits are fine (they go out with the release). Non-patch bumps take a flag, so run the script directly:
`.\scripts\release.ps1 -Bump minor` (also `-Bump major`, `-Version 2.0.0`, `-SkipTests`, `-Branch <name>`).
If a build fails after the version was written, the bump is reverted. The zips land in `.output/`
(Chrome store zip, Firefox zip, and the sources zip AMO requires).

- `WXT_NO_LAUNCH=1 pnpm dev` skips the auto-launched browser; load `.output/chrome-mv3` unpacked in a normal Chrome (HMR still connects).
- **UI preview without an extension context** (mock data + in-memory `chrome` shim): build, serve `.output/chrome-mv3` statically, open `sidepanel.html?mock=1`.
- **Run a single DSP test**: `node --test src/features/vocal-reducer/engine/center-cut-dsp.test.ts` (or `node --test --test-name-pattern "WOLA" src/features/vocal-reducer/engine/center-cut-dsp.test.ts`).

### E2E (real browser, measures processed audio output)
```powershell
pnpm dlx @puppeteer/browsers install chrome@stable --path ./.browsers   # once
node e2e/make-tone.mjs ; node e2e/make-stereo-mix.mjs                    # once, generates WAV fixtures
pnpm wxt build --mode testing   # `testing` mode grants <all_urls> host perms so no native prompts block the run
node e2e/run.mjs                # add --headful to watch
node e2e/library.mjs            # background library + sync integration (pnpm test:e2e:library)
```
The harness plays a 440 Hz tone and asserts on the **processed output** (e.g. 880 Hz after +12 st) via `window.__noteByNoteDebug` in the content script and `window.__panelDebug` in the side panel.

## Architecture

This is a **multi-context extension**. The single most important structural fact: **the audio engine lives in the page (content script), not in the side panel.** The side panel is a thin UI mirror that connects to the engine over a typed `chrome.runtime` Port. This is why practice flows (loops, sequences, playback) survive the side panel closing.

### Source layout (vertical feature slices)
The tree is organized by **feature**, not by layer:
- **`src/core/`** — shared platform: `engine/` (controller, media-engine/-detect, attach-audio), `audio/` (pipeline, fft, silence-detector), `messaging/` (protocol shell, ports, rpc), `model/` (shared types + defaults + format + track-identity + thumbnail), `persist/` (library, backup, migration), and `state/` (session, library, track-sync, connect, view).
- **`src/features/<feature>/`** — one folder per product feature (chords, pitch, speed, vocal-reducer, eq, loops, markers, snippets, count-in, library, sync, settings, shortcuts), each with an `engine/` subfolder (content-script code: worklets, schedulers, DSP factories) and/or a `panel/` subfolder (side-panel stores + components), plus optional `protocol.ts` (its wire-message fragment). **`engine/` and `panel/` never cross-import**, so the content and panel bundles stay separate.
- **`src/ui/`** — shared/presentational UI (Workspace, Panel, PanelStack, Timeline, chrome bars, `shared/` primitives, icons, dismiss).
- **`src/dev/`** — preview-only helpers (`browser-shim`, `mock`).
- **`src/entrypoints/`** — thin WXT composition roots (unchanged location).

**Dependency direction:** `entrypoints → core composition roots (pipeline, controller, protocol, App, track-sync) → features → core primitives (model, messaging, audio/fft, ui)`. Composition roots wire feature behavior directly; features never import the orchestrators. Domain types stay central in `core/model/types.ts` (the shared engine↔panel contract).

### Execution contexts (`src/entrypoints/`)
- **`sidepanel/`** — the Svelte UI. Holds no engine state of its own; mirrors the active tab's engine.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patrickiel/note-by-note](https://github.com/patrickiel/note-by-note) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
