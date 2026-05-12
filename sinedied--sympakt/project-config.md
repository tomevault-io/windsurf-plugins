---
trigger: always_on
description: A single-page application for managing sample packs for the Elektron Syntakt synthesizer. Built as a client-only web app — all audio processing happens in the browser.
---

# Sympakt

A single-page application for managing sample packs for the Elektron Syntakt synthesizer. Built as a client-only web app — all audio processing happens in the browser.

## Overview

- **Purpose**: Create, edit, preview, and export 64-slot sample banks compatible with the Elektron Syntakt
- **Architecture**: Frontend-only SPA using Lit web components, Vite build tooling, TypeScript
- **Audience**: Elektron Syntakt owners who want to manage their sample packs from the browser
- **Deployment**: GitHub Pages (static site)

## Project Structure

```
src/
├── assets/
│   └── PressStart2P-Regular.woff2  # Pixel font (inlined at build time)
├── components/       # Lit web components
│   ├── app-shell.ts        # Main application shell
│   ├── sample-bank.ts      # 64-slot scrollable bank with drag & drop
│   ├── sample-editor.ts    # Destructive sample editor modal (trim, FX, slicer)
│   ├── sample-slot.ts      # Individual sample slot
│   ├── waveform-view.ts    # Pixelated waveform preview canvas
│   ├── virtual-keyboard.ts # 2-octave chromatic keyboard for sample auditioning
│   └── export-dialog.ts    # Export options dialog
├── services/
│   ├── audio-effects.ts    # Audio DSP: trim, reverse, fade, normalize, gain, bitcrush, filter
│   ├── audio-engine.ts     # Web Audio API: decode, resample, preview, analyze
│   ├── persistence.ts      # IndexedDB session persistence (bank + settings)
│   ├── transient-detection.ts # Transient onset detection & even slicing utilities
│   ├── wav-encoder.ts      # Encode PCM data to 16-bit/48kHz/mono WAV
│   ├── wav-decoder.ts      # Decode WAV files
│   └── zip-service.ts      # ZIP import/export using fflate
├── state/
│   └── bank-state.ts       # Reactive state management for the sample bank
├── types/
│   └── index.ts            # Shared TypeScript types and interfaces
├── styles/
│   └── theme.ts            # Global styles, Elektron design tokens, pixel font
├── icons.ts                # SVG icon library (Lit svg templates)
├── vite-env.d.ts           # Vite client type declarations
├── index.ts                # App entry point
└── index.html              # HTML shell
```

## Key Technologies and Frameworks

- **Runtime**: Node.js 24+, ESM modules
- **Language**: TypeScript (strict mode)
- **Build**: Vite 6+ with `vite-plugin-singlefile` (single HTML output)
- **UI**: Lit 3+ web components (no framework)
- **Audio**: Web Audio API (decoding, resampling, playback, waveform analysis)
- **ZIP**: fflate (lightweight, zero-dependency compression)
- **Styling**: CSS via Lit `css` tagged templates; Elektron-inspired dark theme with pixel fonts
- **Icons**: Inline SVG via Lit `svg` tagged templates (`src/icons.ts`)

## Constraints and Requirements

- **No backend** — everything runs client-side
- **Single-file build** — production build outputs a single self-contained `index.html` (all JS, CSS, fonts, and favicon inlined)
- **Minimal dependencies** — prefer browser APIs over libraries
- **Export format**: 16-bit, 48kHz, mono WAV (Syntakt requirement)
- **Normalization**: peak normalization applied per-sample on export (enabled by default, can be toggled in export dialog)
- **Max sample length**: looped samples export only the loop region; non-looped samples are truncated to 5 seconds (10 seconds in LOFI mode)
- **Max loop duration**: 5 seconds (10 seconds in LOFI mode)
- **Bank size**: exactly 64 slots
- **File naming on export**: `<slot_number>_<sample_name>[_<detected_note>].wav` (e.g., `01_kick.wav`, `10_Kick_C3.wav`). Dual split slots use: `<slot_number>_<name_A>-<name_B>_DUAL.wav`
- **Metadata**: JSON file included in exported ZIP with original filenames, sample options, loop settings, detected notes, and structure
- **Audio buffer preservation**: full audio duration is kept in memory (no truncation at import); truncation/extraction happens only at export time

## Development Workflow

```bash
# Install dependencies
npm install

# Start dev server
npm run dev

# Type check
npm run typecheck

# Build for production
npm run build

# Preview production build
npm run preview
```

## Coding Guidelines

- Use Lit reactive properties and decorators for component state
- Prefer `@property()` for public API, `@state()` for internal state
- Use TypeScript strict mode; avoid `any`
- Components should be self-contained with scoped styles via Shadow DOM
- Use `css` tagged template literals for styles — no external CSS files
- Keep audio processing in dedicated services, not in components
- Use `async`/`await` for all asynchronous operations
- Name custom elements with `sp-` prefix (e.g., `<sp-sample-slot>`)
- Use named exports; one primary export per file
- Barrel exports from `types/index.ts` only

## Audio Processing Notes

- Decoding: use `AudioContext.decodeAudioData()` for broad format support
- Resampling: use `OfflineAudioContext` at 48kHz to resample imported audio (full duration preserved)
- Waveform generation: compute RMS values per pixel column from decoded PCM data
- Truncation: non-looped samples > 5s (or > 10s in LOFI, > 20s in XLOFI, > 40s in SXLOFI, > 80s in GXLOFI mode) show orange truncated region on waveform; truncation happens at export

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sinedied/sympakt](https://github.com/sinedied/sympakt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
