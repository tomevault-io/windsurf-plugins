---
trigger: always_on
description: `@gfazioli/mantine-audio` — A Mantine-native audio player for React with **waveform visualisation** and **live spectrum analyser**, built on the Web Audio API. Compound component API (`<Audio.Controls>`, `<Audio.PlayButton>`, `<Audio.Timeline>`, `<Audio.Waveform>`, `<Audio.Spectrum>`, …), fully headless `useAudio` hook, theme-aware styling, accessibility, and full Styles API support.
---

# CLAUDE.md

## Project

`@gfazioli/mantine-audio` — A Mantine-native audio player for React with **waveform visualisation** and **live spectrum analyser**, built on the Web Audio API. Compound component API (`<Audio.Controls>`, `<Audio.PlayButton>`, `<Audio.Timeline>`, `<Audio.Waveform>`, `<Audio.Spectrum>`, …), fully headless `useAudio` hook, theme-aware styling, accessibility, and full Styles API support.

Bootstrapped from `mantine-base-component` (the GitHub template for the Mantine Extensions ecosystem).

## Commands

| Command | Purpose |
|---------|---------|
| `yarn build` | Build the npm package via Rollup |
| `yarn dev` | Start the Next.js docs dev server (port 9281) |
| `yarn test` | Full test suite (syncpack + oxfmt + typecheck + lint + jest) |
| `yarn jest` | Run only Jest unit tests |
| `yarn docgen` | Generate component API docs (docgen.json) |
| `yarn docs:build` | Build the Next.js docs site for production |
| `yarn docs:deploy` | Build and deploy docs to GitHub Pages |
| `yarn lint` | Run oxlint + Stylelint |
| `yarn format:write` | Format all files with oxfmt |
| `yarn storybook` | Start Storybook dev server |
| `yarn clean` | Remove build artifacts |
| `yarn release:patch` | Bump patch version and deploy docs |

> **Important**: After changing the public API (props, types, exports), always run `yarn clean && yarn build` before `yarn test`, because `yarn docgen` needs the fresh build output.

## Architecture

### Workspace Layout

Yarn workspaces monorepo with two workspaces: `package/` (npm package) and `docs/` (Next.js documentation site).

### Package Source (`package/src/`)

- `Audio.tsx` — Main component using `factory()` with Mantine's Styles API. Wraps a native `<audio>` element with React-friendly props (controlled `playing`/`currentTime`/`volume`/`playbackRate`), 4 variants (`overlay`/`minimal`/`floating`/`bordered`), keyboard shortcuts, `asBackground` preset.
- `Audio.module.css` — CSS module with custom properties and data-attribute selectors
- `Audio.test.tsx` — Jest tests using `@mantine-tests/core` render helper
- `Audio.story.tsx` — Storybook stories
- `use-audio.ts` — Headless `useAudio` hook returning state + actions + Web Audio context. Handles event listeners on the `<audio>` element, decodes peaks via `decodeAudioData`, lazy-creates the `AudioContext` + `MediaElementAudioSourceNode` + `AnalyserNode` on first play.
- `Audio.context.ts` — Internal context shared with compound sub-components
- `captions.ts` — Pure helpers deriving caption state from a `TextTrackList` (`getCaptionTracks`, `isCaptionsActive`, `readActiveCueText`). Kept free of the DOM on purpose: jsdom stubs the TextTrack API, so this is the only way the cue logic gets real test coverage (see Testing).
- `components/` — Twelve compound sub-components:
  - **Core**: `AudioPlayButton`, `AudioMuteButton`, `AudioSkipButton`, `AudioTimeDisplay`, `AudioTimeline`, `AudioControls`
  - **Extras**: `AudioVolumeSlider`, `AudioSpeedControl`, `AudioWaveform`, `AudioSpectrum`
  - **Captions** (1.1.0): `AudioCaptions`, `AudioCaptionsButton`
- `index.ts` — Public exports (root component + sub-components + hook + types)

### Web Audio API integration

Two distinct flows:

1. **Waveform peaks** (`AudioWaveform`): when `src` changes the hook `fetch()`es the file, calls `decodeAudioData()` on a one-shot `AudioContext`, then downsamples to `waveformSamples` peaks (default 512). The decoded peaks are exposed via `ctx.peaks: Float32Array | null`. CORS-failing decodes produce `ctx.peaksError` and a null `peaks` — the Waveform component degrades gracefully.

2. **Live spectrum** (`AudioSpectrum`): the *main* `AudioContext` + `AnalyserNode` are lazy-created on first `play()` (browsers throw if you create them too early). The `<audio>` element is connected via `createMediaElementSource`, then to the `AnalyserNode`, then to `destination`. `AudioSpectrum` reads `analyser.getByteFrequencyData()` in a `requestAnimationFrame` loop while playing, and decays to zero when paused.

> **CORS**: the `<audio crossOrigin="anonymous">` attribute is mandatory for remote files to be decoded by Web Audio. The default `defaultProps.crossOrigin = 'anonymous'` covers most CDN cases.

### Build Pipeline

Rollup bundles to dual ESM (`dist/esm/`) and CJS (`dist/cjs/`) with `'use client'` banner. CSS modules are hashed with `hash-css-selector` (prefix `me`). TypeScript declarations via `rollup-plugin-dts`. CSS is split into `styles.css` and `styles.layer.css` (layered version).

### Docs (`docs/`)

- `docs/data.ts` — Package metadata
- `docs/docs.mdx` — Main documentation content
- `docs/demos/` — Interactive demos using `@mantinex/demo`
- `docs/pages/index.tsx` — Assembles Shell, PageHeader, DocsTabs, and the MDX content
- `docs/styles-api/` — Styles API data for the documentation table
- `docs/docgen.json` — Auto-generated from TypeScript types (don't edit manually)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gfazioli/mantine-audio](https://github.com/gfazioli/mantine-audio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
