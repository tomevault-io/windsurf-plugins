---
trigger: always_on
description: Project map for agents. Goal: skip the exploration phase and go straight to fixing bugs in Hover Zoom+.
---

# AGENTS.md

Project map for agents. Goal: skip the exploration phase and go straight to fixing bugs in Hover Zoom+.

This file is a **living document**: keep it always correct and make it more useful with every session.

- After each session, fold back what you learned into the matching section, in the same change as the code that taught it: non-obvious root-cause mechanics, surprising behavior, new gotchas/invariants, better reproduction or debugging recipes.
- When code or architecture changes (moved/renamed files, functions, DOM ids, options, flows, tooling), update the affected sections immediately — a stale document is worse than none.
- Keep it terse and durable: stable names instead of line numbers, facts/invariants/recipes instead of narration. Correct or delete what is no longer true instead of appending contradictions; resist letting it grow into a changelog.

## What this is

Chrome MV3 extension ("Hover Zoom+") that shows a floating, zoomed preview (image/video/audio) next to the cursor while hovering links/thumbnails. Plain classic-script JavaScript: **no build step, no npm, no node_modules, no test suite**. Everything runs as content scripts in the page's isolated world.

## Repo layout

- `js/hoverzoom.js` — all core behavior (~5k lines, single closure). Hover detection, viewer creation/positioning, media loading, key/mouse/wheel handling, galleries, video/audio/HLS playback. Most bugs are fixed here.
- `js/common.js` — default `options` object (single source of truth for option names/defaults), key-binding defaults, shared helpers used by content script, background, options and popup pages.
- `js/background.js` — MV3 service worker (permissions, history, downloads, messaging).
- `js/tools.js`, `js/plugins.js`, `js/findMatchingBracket.js` — small shared helpers loaded before `hoverzoom.js`.
- `plugins/*.js` — one small script per supported site; each registers into the global `hoverZoomPlugins` list and resolves full-size media URLs for its site. `plugins/default.js` is always loaded (generic fallback). Per-site plugins are injected via their own `content_scripts` entries in `manifest.json` (matched by domain; several plugins can share a domain, e.g. reddit).
- `html/options.html` + `js/options.js`, `html/popup.html` + `js/popup.js` — options UI and toolbar popup. Control ids mirror option names (`chk*`, `txt*`, `sel*`, `rng*`).
- `_locales/*/messages.json` — all UI strings; option labels/tooltips are `opt*` keys. **Read `opt*` in `_locales/en/messages.json` to learn an option's real semantics** — several are negated (e.g. `optDisableMouseWheelForVideo` = "don't use mousewheel to navigate within video"; checked disables seeking).
- `js/hls.js` — vendored hls.js (webpack bundle) for HLS/DASH video playlists. `js/jszip.js` vendored for the pixiv plugin. `js/jquery-3.7.1.js`, `js/gumby.js` (options-page UI framework) vendored too.
- `manifest.json` — MV3, min Chrome 130. Core content script = jquery + tools + common + plugins + `plugins/default.js` + hoverzoom + hls + findMatchingBracket, on `<all_urls>`, `all_frames: true`. A few sites need page-world shims exposed as web-accessible resources (`js/hoverZoomXHROpen.js`, `js/duckduckgoInjected.js`, `js/hoverZoomVintedFetch.js`).
- `release.py` / `release.cmd` — packaging. `CHANGELOG.md` is **auto-generated** by github_changelog_generator — never hand-edit it.

## Core architecture (`js/hoverzoom.js`)

One big closure (`loadHoverZoom()`) with shared mutable state. Variables you actually need to know:

- `imgFullSize` — jQuery of the displayed media element (`<img>` or `<video>`); `null` = nothing displayed.
- `srcDetails` — current source: `url`, `audioUrl`, `subtitlesUrl`, `video`/`playlist`/`audio` flags, `naturalWidth`/`naturalHeight`, `naturalSrc` (the source the natural dims were sampled for).
- `hls` — hls.js instance for playlist playback (module scope, shared).
- `viewerLocked`, `zoomFactor` — lock/zoom mode state; `loading`; key-state flags (`actionKeyDown`, `fullZoomKeyDown`, `hideKeyDown`, `plusKeyDown`/`minusKeyDown`/`arrowUp/DownKeyDown`, `zoomSpeedFactor`).
- `currentSrcToken` — bumped when a new source is scheduled so an in-flight viewer close (fade-out) cannot wipe the new source's state. Keep this invariant wherever a source is scheduled.
- `lastScrollTime` + `options.scrollWheelCooldown` — wheel throttle.

Lifecycle (memorize this chain):

1. `prepareImgLinks` / `prepareImgLinksAsync` — scan DOM, run plugins, tag hoverable elements (MutationObserver keeps up with dynamic pages).
2. `documentMouseMove` — finds `.hoverZoomLink` in the target's ancestry, schedules `loadFullSizeImage` after `displayDelay` / `displayDelayVideo`; while a preview is displayed it just calls `posViewer()`.
3. `loadFullSizeImage` — creates the media element (`<video>`, hidden `<audio>` + spectrogram `<img>`, HLS playlist via hls.js, or `<img>`).
4. Media events (`loadedmetadata`, `loadeddata`) → `displayFullSizeImage()` builds the viewer DOM and fades it in.
5. `posViewer()` — ALL sizing and positioning; re-run on every mousemove over the same link.

Viewer DOM (built fresh by `displayFullSizeImage`, which empties `#hzViewer` and recreates its children):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [extesy/hoverzoom](https://github.com/extesy/hoverzoom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
