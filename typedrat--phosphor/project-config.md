---
trigger: always_on
description: A physically-based X-Y CRT simulator in Rust using wgpu for GPU compute/rendering and egui for the UI. The goal is to accurately model the physics of phosphor emission, decay, and CRT electron beam behavior — not just approximate the visual appearance.
---

# Phosphor — X-Y CRT Simulator

## Project Overview

A physically-based X-Y CRT simulator in Rust using wgpu for GPU compute/rendering and egui for the UI. The goal is to accurately model the physics of phosphor emission, decay, and CRT electron beam behavior — not just approximate the visual appearance.

## Build & Run

```bash
cargo run              # Debug build
cargo run --release    # Release build (recommended for performance)
RUST_LOG=debug cargo run  # With debug logging
```

## Architecture

### Core Rendering Pipeline (GPU)

Accumulation buffer architecture — the phosphor screen state lives entirely on the GPU as a flat storage buffer of scalar energy layers. Each frame the pipeline runs these passes in order:

1. **Beam Write Pass** (compute shader): Splats new beam hits as Gaussian spots, depositing scalar energy into accumulation layers per decay term. Tier-1 instantaneous terms are folded into a one-frame scalar layer.
2. **Spectral Resolve Pass** (fragment shader): Reads accumulation layers, applies per-group emission weights, integrates spectral energy via CIE weights to produce XYZ → linear sRGB into the HDR buffer. Runs _before_ decay so newly deposited energy is visible at full brightness.
3. **Decay Pass** (compute shader): Applies tier-aware decay to the accumulation buffer — multiplicative `exp(-dt/τ)` for slow exponentials (tier 2), elapsed-time tracking for power-law terms (tier 3), and clears tier-1 instant layers.
4. **Faceplate Scatter Pass** (compute shader): Downsample HDR → half-resolution, then separable Gaussian blur (horizontal + vertical) for halation/bloom.
5. **Composite Pass** (fragment shader): Combines HDR + faceplate scatter → applies glass tint, curvature, edge falloff, tonemapping → final display output.
6. **egui Overlay Pass** (render pass): Side panel / controls drawn on top of the CRT output.

### Accumulation Buffer

A flat `wgpu::Buffer` of `u32` values (bitcast to/from `f32`) indexed as `layer * (width * height) + y * width + x`. The number of layers is variable per phosphor, determined by decay term classification:

- **Tier 2 layers**: One scalar layer per slow exponential term
- **Tier 3 layers**: 2 layers if power-law present (peak energy + elapsed time)
- **Tier 1 layer**: 1 layer if instantaneous terms present (cleared each frame after spectral resolve)

### Spectral Representation

- `SPECTRAL_BANDS` (compile-time constant, 16) bands at ~25nm intervals across 380–780nm
- Changing this constant propagates through the entire pipeline (Rust const + WGSL pipeline-overridable constant)
- Emission weights are stored per emission group in `SpectralResolveParams`, not per accumulation layer — the accumulation buffer stores scalar energy, and spectral weighting happens at resolve time
- CIE 1931 XYZ integration weights are pre-computed per band

### Phosphor Model

Each phosphor type (P1, P2, P7, P11, P31, etc.) is defined by:

- Per-layer spectral emission curve (normalized weights across `SPECTRAL_BANDS`, Gaussian approximation from peak wavelength and FWHM)
- Decay terms: a `Vec<DecayTerm>` with up to 8 terms, each either `Exponential { amplitude, tau }` or `PowerLaw { amplitude, alpha, beta }`
- Dual-layer support for phosphors with distinct fluorescence/phosphorescence (P2, P7, P14, etc.)
- Peak wavelength, relative luminance, relative writing speed

Phosphor data is baked at compile time from `data/phosphors.toml` via a proc macro (`phosphor-data-macro`), with runtime loading also supported.

### Three-Tier Hybrid Decay Model

Based on Kuhn (2002) PMT measurements and Selomulya (2003) kinetic data:

| Tier | Condition              | Mechanism                               | Implementation                                                        |
| ---- | ---------------------- | --------------------------------------- | --------------------------------------------------------------------- |
| 1    | τ < 100µs              | Instantaneous (decays within one frame) | Integrated analytically during beam write, stored as one-frame scalar |
| 2    | τ ≥ 100µs, exponential | Slow multiplicative decay               | `value *= exp(-dt/τ)` per frame in accumulation buffer                |
| 3    | Power-law              | Bimolecular DAP recombination           | Tracks per-texel elapsed time, evaluates `peak * (α/(t+α))^β`         |

### Beam Model

- Gaussian core + halo spot profile: `I(r) = (1-h)·exp(-r²/2σ²) + h·exp(-r²/2σ_halo²)`
- Spot size affected by focus setting, acceleration voltage, and beam current (space charge)
- Arc-length resampling decouples energy deposition from input sample rate — consecutive samples are merged to ~0.5× beam sigma spacing for uniform trace brightness

### Input Modes

All input modes produce a common `BeamSample { x, y, intensity, dt }` stream:

1. **Oscilloscope**: Built-in signal generators (sine, triangle, square, sawtooth, noise) for X/Y channels
2. **Audio**: Stereo audio file where L=X, R=Y (for oscilloscope music). Uses symphonia for decoding.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/typedrat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
