---
trigger: always_on
description: Generates a Svelte Playground link with the provided code.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Buzzer Studio is a web-based IDE built with **Svelte 5** for embedded audio development. It converts audio into formats playable on microcontrollers (Arduino, ESP32, etc.) using buzzers and GPIO pins. Five main tools:

1. **Sound Effect Generator** - Creates 1-bit GPIO toggle sequences for retro sound effects
2. **MIDI Converter** - Converts MIDI music files to C/C++ code for buzzer playback
3. **ADPCM Converter** - Encodes audio to 2-bit ADPCM with 4:1 compression
4. **LPC Encoder** - Encodes WAV audio files to LPC (Linear Predictive Coding) speech synthesis
5. **LPC Player** - Plays LPC-encoded speech data using TMS5220/TMS5100 algorithm

## Essential Commands

### Development
```bash
npm run dev              # Start dev server
npm run build            # Build for production (includes type check)
npm run type-check       # Type check without building
```

### Testing
```bash
npm run test             # Run all tests once
npm run test:watch       # Run tests in watch mode
npm run test:ui          # Open Vitest UI
npm run test:coverage    # Generate coverage report
```

### Code Quality
```bash
npm run lint             # Lint TypeScript files (max-warnings 0)
npm run lint:fix         # Auto-fix linting issues
npm run format           # Format code with Prettier
npm run format:check     # Check formatting without modifying
npm run ci               # Full CI: type-check + lint + format:check + test + build
```

## Architecture

### Svelte 5 Component-Based Architecture
The app is built with **Svelte 5** using the new **runes API** (`$state`, `$derived`, `$effect`).

**Key Structure:**
- `src/main.ts` - Mounts the root Svelte app using `mount(App, { target })`
- `src/lib/App.svelte` - Root component with reactive tab navigation
- `src/lib/tools/` - Five tool components (`.svelte` files)
- `src/lib/shared/` - Reusable UI components (RangeControl, Button, etc.)
- Core logic modules (soundEngine, midiConverter, player, visualizer) remain unchanged

### Svelte Component Pattern
Each tool is a `.svelte` component that:
1. Uses `$state` for reactive state management
2. Uses `$derived` for computed values
3. Uses `$effect` for side effects (canvas drawing, audio playback)
4. Imports and uses existing TypeScript modules (soundEngine.ts, midiConverter.ts, etc.)

To add a new tool:
1. Create `src/lib/tools/NewTool.svelte`
2. Import in `src/lib/App.svelte`
3. Add to the tabs array in App.svelte

### Core Module Relationships

**Sound Effect Generator (`SoundEffects.svelte`):**
- Uses `soundEngine.ts` (audio engine) and `presets.ts` (preset data)
- Reactive UI with `$state` for parameters
- Generates 1-bit GPIO toggle sequences using Web Audio API
- Exports to C arrays or Python code

**MIDI Converter (`MidiConverter.svelte`):**
- Uses `midiConverter.ts` (core algorithm), `player.ts` (playback), `visualizer.ts` (canvas)
- Canvas visualization via Svelte actions (`use:initCanvas`)
- Polyphonic-to-monophonic splitting algorithm
- Uses `@tonejs/midi` library for MIDI parsing

**ADPCM Converter (`AdpcmConverter.svelte`):**
- Self-contained encoding/decoding implementation
- Uses `$effect` for canvas waveform visualization
- Web Audio API for audio resampling and playback
- 2-bit ADPCM encoding with 4:1 compression

**LPC Encoder (`LpcEncoder.svelte`):**
- Uses `lpcEncoder.ts` which orchestrates 12-stage pipeline in `src/encoder/`:
  1. `wavParser.ts` - Parse WAV file headers and audio data
  2. `audioPreprocessor.ts` - Normalize, downsample, apply pre-emphasis filter
  3. Frame windowing - Split into 25ms overlapping frames
  4. `autocorrelator.ts` - Calculate autocorrelation coefficients
  5. `reflector.ts` - Levinson-Durbin algorithm for reflection coefficients
  6. `pitchEstimator.ts` - Detect voiced/unvoiced frames and pitch
  7. RMS energy calculation
  8. `closestValueFinder.ts` - Quantize parameters to TMS5220 tables
  9. Repeat detection - Find and mark repeated frames
  10. `binaryEncoder.ts` - Pack parameters into bit patterns
  11. `hexConverter.ts` - Convert to hex strings
  12. Generate C/Python code for TMS5220 chip

**LPC Player (`LpcPlayer.svelte`):**
- Uses `talkieStream.ts` for TMS5220/TMS5100 speech synthesis
- Plays LPC-encoded hex data via Web Audio API
- Includes sample phrases from Talkie library

### Critical Implementation Details

**Hardware-Accurate Timing:**
The same timing calculations are used for both browser audio preview AND microcontroller code export. This ensures what you hear in the browser matches what the hardware produces - exact to the microsecond. See `soundEngine.ts::exportBitTimings()`.

**Polyphonic-to-Monophonic Splitting:**
The most complex algorithm in the codebase. Splits MIDI files with multiple simultaneous notes into multiple monophonic streams (one tone per stream) that microcontroller buzzers can play. See `midiConverter.ts::splitIntoMonophonicStreams()`.

**Svelte 5 Reactivity:**
- **`$state`** - All reactive variables (params, file data, UI state)
- **`$derived`** - Computed values (showContent, compressionRatio, etc.)
- **`$effect`** - Side effects for canvas drawing, audio visualization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atomic14/ch32v003-audio](https://github.com/atomic14/ch32v003-audio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
