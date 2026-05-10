---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Guillotine is a JUCE-based audio plugin implementing a clipping effect with animated guillotine visualization. Supports VST3/AU on macOS, VST3 on Windows, and VST3/LV2/CLAP on Linux. Built by Dichotic Studios.

## Build Commands

```bash
./scripts/build.sh              # Build Release, install to /Library/Audio/Plug-Ins/ (requires sudo)
./scripts/build.sh debug        # Debug build
./scripts/build.sh clean        # Clean build artifacts
./scripts/build.sh --no-install # Build without installing
./scripts/standalone.sh         # Quick UI preview - builds standalone app and launches it
./scripts/watch.sh              # Auto-reload: watches src/, web/ and rebuilds on change
./scripts/validate.sh           # Run pluginval at strictness 10 (or pass level: ./scripts/validate.sh 5)
```

**Build system:** CMake with Xcode generator. First build runs `cmake -B build -G Xcode` automatically.

Build outputs: `build/Guillotine_artefacts/Release/VST3/Guillotine Clip.vst3` and `AU/Guillotine Clip.component`

## Testing

```bash
# Setup (one time)
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# Run all tests
pytest tests/ -v

# Run specific test file
pytest tests/integration/test_integration.py -v
```

Test types:
- `tests/clipper/` - Clipper DSP tests (delta, hard clip, oversampling, intersample, smoothing)
- `tests/integration/` - Plugin-level tests (gain, bypass, NaN defense, invariance)
- `tests/compliance/` - pluginval DAW compatibility checks
- `tests/unit/` - C++ unit tests (CMake-based, Catch2): clipper, oversampler, envelope buffer, transient

## Architecture

**DSP Layer:**
- `src/PluginProcessor.cpp` - Audio processing, parameter management
- `src/dsp/ClipperEngine.cpp` - Main DSP chain: gain → M/S → oversample → clip → downsample
- `src/dsp/Clipper.cpp` - Saturation curve implementations (hard, tanh, arctan, etc.) with parameter smoothing
- `src/dsp/Oversampler.cpp` - Polyphase oversampling with deferred rebuild for thread safety
- `src/dsp/StereoProcessor.cpp` - M/S encoding, stereo link
- `src/dsp/EnvelopeBuffer.h` - Thread-safe ring buffer for waveform visualization

**UI Layer (WebView):**
- `src/PluginEditor.cpp` - JUCE WebBrowserComponent, serves web resources via `getResource()`
- `web/index.html` - Entry point, loads main.js
- `web/main.js` - App initialization, JUCE parameter binding
- `web/lib/juce-bridge.js` - C++/JS communication bridge

**Web Components:**
- `web/components/controls/` - knob.js, lever.js, toggle.js (interactive controls)
- `web/components/display/` - waveform.js, digits.js, blood-pool.js (visualizations)
- `web/components/views/` - guillotine.js, microscope.js (main views)
- `web/lib/` - Utilities: config.js, theme.js, guillotine-utils.js, saturation-curves.js

**Assets:**
- `web/assets/` - All images: guillotine graphics, toggles, numeric digits, text labels, textures, fonts

**DSP Chain (ClipperEngine.cpp):**
```
Input → InputGain → M/S Encode → Upsample → Clipper → Downsample → M/S Decode → EnforceCeiling → OutputGain → Delta Monitor → Output
       ↓                                                                                                              ↓
       └──────────────────────────────── Dry (matched oversample) ──────────────────────────────────────────────→ Mix → Output
```

**Parameters:**
| Parameter | Range | Notes |
|-----------|-------|-------|
| curve | 0-6 | Hard, Tanh, Atan, Quint, Cubic, Knee, T2 |
| curveExponent | 1.0-4.0 | Smoothed (2ms ramp) |
| oversampling | 0-5 | 1x/2x/4x/8x/16x/32x |
| inputGain | -24 to +24 dB | |
| outputGain | -24 to +24 dB | |
| ceiling | -24 to 0 dB | Smoothed, linked to blade position |
| filterType | 0-1 | Linear phase / Min phase |
| stereoMode | 0-1 | L/R / M/S |
| enforceCeiling | bool | Hard limit on output |
| deltaMonitor | bool | Output clipped signal only |
| dryWet | 0-100% | Phase-coherent mixing |
| gainMode | 0-2 | Manual / Match / Maximize (default: Match) |
| bypass | bool | Blade up/down |

**Gain Compensation (Match Mode):**

Match mode auto-compensates output gain so clipped audio stays at roughly the same perceived loudness. It runs two reference signals through the current curve/ceiling/exponent and measures RMS loss:

- **Transient reference:** `exp(-8t)` — exponential decay, CF ≈ 12dB. Models drum/percussion content where only the peak tip gets clipped.
- **Tonal reference:** `exp(-25.5*(t-0.5)²)` — Gaussian bell, CF ≈ 6dB. Models sustained content (guitars, synths, vocals) where more energy sits near the ceiling.

The two compensation values are blended based on ceiling depth:
- At -6dB ceiling → pure transient (less compensation, because transient content loses less from clipping)
- At -18dB ceiling → pure tonal (more compensation, because deep clipping removes more sustained energy)
- Between -6 and -18 → linear interpolation

Additional adjustments:
- **Progressive reduction:** -2dB linear ramp from 0dB ceiling to -60dB ceiling (prevents over-compensation at extreme settings)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noahbaxter/guillotine](https://github.com/noahbaxter/guillotine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
