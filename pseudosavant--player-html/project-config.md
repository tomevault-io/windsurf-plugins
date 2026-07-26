---
trigger: always_on
description: This repo ships a single-file web app. Source is split under `src/` and assembled into `dist/player.html` by a tiny build script (no bundler).
---

# AGENTS.md

This repo ships a single-file web app. Source is split under `src/` and assembled into `dist/player.html` by a tiny build script (no bundler).

## Project summary
- `player.html` is a drop-in audio/video player for HTTP directory listings.
- It treats the directory listing HTML as an API (via an iframe parser) and renders folders + media files as tiles.
- Everything (HTML, CSS, JS, SVG icons, PWA manifest) is inlined.

## Key files and folders
- `src/player.html`: Dev template (references CSS/JS/SVG sources).
- `src/styles.css`: All styles (inlined at build time).
- `src/js/*.js`: JS modules, inlined in order by the build.
- `src/svg/*.svg`: SVG sprite pieces, concatenated into `.xlinks`.
- `src/assets/*`: Binary assets inlined as data URIs at build time.
- `dist/player.html`: Generated single-file distributable.
- `build.py`: Build + `--watch`.
- `README.md`: Usage and features overview.
- `assets/`: Only for docs/screenshots; not used at runtime.
- `videos/`: Sample media files.

## HTML structure (high level)
`<body class="no-subtitles is-stopped">`
- `.player-container`
  - `.media-container`
    - `<video.player>`
    - `.media-title`
    - `.fileinfo.modal` (metadata list)
    - `.help.modal` (keyboard shortcuts list)
    - `.subtitle-selection.modal`
    - `.settings.modal`
    - `.modal-background-overlay`
  - `.controls`
    - `.progress-bar` (buffer/progress)
    - `.trick-container` + `<video.trick>` (hover preview)
    - `.controls-settings` (info/settings buttons)
    - `.primary-buttons` (play/pause, seek, volume, fullscreen, pip, subtitles, playback rate)
    - `.controls-sources` (cloud/local file pickers)
    - `.current-timestamp`
- `.links` (folder + file tiles)
- `<footer>` (version link)
- `.xlinks` (inline SVG sprite sheet)

## CSS architecture
- Source lives in `src/styles.css`; build inlines it into `<style primary>`.
- Heavy use of CSS custom properties for theme and sizing:
  - `--theme-hue`, `--theme-color`, `--tile-*`, `--progress-bar-*`, etc.
  - Light/dark values set via `data-color-scheme` and `prefers-color-scheme`.
- Layout is CSS Grid for page + controls, and flex/grid for tiles.
- UI state is driven by body/html classes:
  - Playback: `is-loaded`, `is-playing`, `is-paused`, `is-stopped`
  - Media: `is-audio`, `is-pip`
  - Capabilities: `no-volume`, `no-subtitles`
  - UX: `no-thumbnail-animation`, `fadeout` (controls autohide)
- SVG icons are embedded and referenced with `<use xlink:href="#svg-...">`.

## JavaScript modules (in order)
Each module sets `const global = window` and exports to `global` when needed. Build inlines modules in template order.

1) Global config
   - Defines `window.app`:
     - `options.cloud` (OneDrive/GDrive credentials)
     - `options.thumbnails` (timestamps, size, mime, cache, resizeQuality, concurrency)
     - `options.updateRate` (timeupdate, trickHover)
     - `links`, `metadata` placeholders

2) Utilities
   - DOM helper `$()` (single element, jQuery-like methods)
   - `delay`, `throttle`
   - `storageStore/Retrieve/Remove`
   - `secondsToHMS`, `secondsToString`, `pad`, `addCommas`
   - Type helpers: `isNumber`, `isString`, `isBoolean`, `isUndefined`
   - CSS var helpers: `setCSSVariableString/Number`, `clearCSSVariable`, `getCSSVariable`
   - UTF-safe base64 helpers: `base64EncodeUTF`, `base64DecodeUTF`

3) PWA
   - Builds a data-URI manifest at runtime (icon comes from the inline favicon).
   - Adds file handlers for audio/video types.

4) Social metadata
   - Injects `og:*`, `twitter:*`, and MS tile metadata based on document title/description/icon.

5) Video thumbnail engine (video-thumbnail.js)
   - Generates data-URI thumbnails from a video URL.
   - Optional localStorage cache with prefix `video-thumbnail.js`.
   - Exports `window.videoThumbnail`.

6) File pickers (cloud)
   - OneDrive and Google Drive picker flows.
   - Only shown when HTTPS and keys are configured.
   - Exports `window.onedrive` and `window.gdrive`.

7) folder.api (directory listing parser)
   - Loads directory listing in an iframe.
   - Detects server type (nginx/apache/iis/deno) and parses DOM for links + metadata.
   - Exports `window.folderApiRequest`.

8) App code
   - Core player UI/behavior:
     - Supported media types computed from `video.canPlayType`.
     - Directory listing -> tiles + thumbnails.
     - Playback controls (play/pause, seek, stop, volume, fullscreen, PiP).
     - Progress bar + trickplay hover.
     - Subtitles (SRT -> VTT conversion, track injection).
     - Settings modal, persistence via localStorage.
     - Shareable hash state (location, media, time).
     - Theme color updates (`--theme-hue`) + poster background.
     - File metadata modal (HEAD/GET headers, bitrate, fps).
   - Initializes in `main()` at end of file.

## Data flow and state
- `app.links` holds parsed folder/file lists from `folderApiRequest`.
- `app.metadata` is updated on `loadedmetadata` and used for info modal + seek deltas.
- `hashState` tracks `{ location, media }`, serialized into `location.hash` via base64 JSON.
- `updateHash()` runs during playback to keep shareable URLs current.

## Settings and persistence

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pseudosavant/player.html](https://github.com/pseudosavant/player.html) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
