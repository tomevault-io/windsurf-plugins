---
trigger: always_on
description: Guidance for Claude Code working in this repository.
---

# CLAUDE.md

Guidance for Claude Code working in this repository.

## Project Overview

Tract Plugin Pack — a Cargo workspace of audio effect plugins (VST3, CLAP, standalone) built with [nih-plug](https://github.com/robbert-vdh/nih-plug) in Rust. Every plugin's GUI is CPU-rendered (softbuffer + tiny-skia + fontdue, no GPU) and freely resizable.

### Plugins

- **wavetable-filter** — wavetable frames as FIR kernels. Raw (direct convolution, zero latency) / Phaseless (STFT magnitude-only, no pre-ringing) modes.
- **miff** — convolution filter whose FIR kernel is hand-drawn with an MSEG editor (sibling of wavetable-filter, but no wavetable file). Raw / Phaseless modes.
- **gs-meter** — loudness meter + gain utility. dB mode (peak, true peak, RMS, crest) and LUFS mode (EBU R128, LRA). Designed for 100+ instances.
- **gain-brain** — gain utility with cross-instance group linking (16 groups, Absolute/Relative, Invert). Inspired by BlueCat Gain Suite.
- **tinylimit** — low-latency wideband peak limiter. Feed-forward + lookahead, dual-stage envelope, soft knee, optional ISP, 7 presets. Inspired by DMG TrackLimit.
- **satch** — detail-preserving spectral saturator. FFT per-bin magnitude saturation preserves quiet components through clipping.
- **six-pack** — six-band parallel "distort the difference" multiband saturator. 6 algorithms, per-band M/S routing, linear-phase oversampling, de-emphasis.
- **pope-scope** — multichannel oscilloscope with beat sync. Shared global store across 16 instances, 3 display modes, hierarchical mipmap ring buffer.
- **warp-zone** — spectral shifter/stretcher (phase vocoder). Shift, Stretch, Freeze, Feedback, frequency range. 4096-pt FFT, 1024 hop, ~85 ms latency.
- **imagine** — multiband stereo imager modeled on iZotope Ozone Imager. 4 bands, Ozone-style Width law, per-band Stereoize, Recover Sides, 4 vectorscope modes.
- **multosis** — 16-row grid sequencer with per-row audio effects driven by MSEGs. 50 effect kinds in the registry (a `None` passthrough plus 49 effects), including a 14-effect Spectral family (`Spectral{Shift,Rotate,Twist,Mirror,Bandpass,Stretch,Scatter,Cascade,Smear,Reverb,Compress,Corrupt,Lofi,Spread}`) sharing `tract_dsp::spectral_engine::SpectralEngine` (audio-thread-safe switchable FFT 512/1024/2048/4096, 50% Hann COLA except `SpectralStretch` which holds its own 75% analyzer for the phase vocoder); `Satch` and `WarpZone` also belong to the Spectral family but run their own fixed-FFT engines (so `EffectKind::family()` reports a 16-member Spectral family).
- **nap** — EDVN velvet-noise character reverb. Three drawn curves (Decay/Width/Tone) over a shared tail-position axis sculpt the tail's loudness, stereo spread, and tonal colour independently. Dual engine: Zero Latency (sparse O(M) tapped-delay convolution, zero reported latency) and Efficient (UPOLS FFT convolution of the baked IR via `PartitionedConvolver`, ~512-sample latency, lower CPU at large/dense settings). Feature-tier (not held to 100+-instance target).
- **hd26** — unison-detune chorus + pseudo-stereo widener, inspired by Serum's Hyper/Dimension. Two on-panel sections labelled `H____` and `D________` (deliberately not branded with Serum's terms; host params use `H `/`D ` prefixes). Chorus section: multi-voice modulated fractional-delay (up to 7 voices, Doppler detune, Szabo-style asymmetric spread, transient-detector Retrig, stereo Width). Widener section: 4-tap pseudo-stereo — AM mode (antisymmetric Side injection → mono-safe) and Pitch mode (delay modulation + cross-feed). Zero latency; no FFT.

## Workspace Structure

Each plugin is a crate (`<plugin>/`). Plus: `tiny-skia-widgets/` (shared CPU-rendered widgets), `tract-dsp/` (shared GUI-free DSP primitives), `docs/` (manuals md+PDF), `xtask/` (build tooling), `nih-plug-widgets/` (legacy vizia widgets, workspace-excluded so its old transitive deps stay out of the lock file).

## Build / Test / Lint

Requires **nightly Rust** (enforced by `rust-toolchain.toml`) for portable SIMD (`std::simd::f32x16`).

```bash
cargo nih-plug bundle <plugin> --release   # VST3 + CLAP bundle
cargo build --bin <plugin> --release       # standalone
cargo build --bin <plugin>                 # debug standalone (GUI testing without DAW)

cargo nextest run --workspace              # all tests (parallel runner)
cargo clippy --workspace -- -D warnings    # lint (CI uses -D warnings)
cargo fmt --check
```

For local release/profile/bundle builds, prefix with `cargo xtask native` (auto-detects host CPU → `-C target-cpu=haswell`). Install the test runner via `cargo install cargo-nextest --locked`; config in `.config/nextest.toml`, CI uses the `ci` profile (retries=1). No doctests. Tests are inline `#[cfg(test)]` modules. Fixtures in `wavetable-filter/tests/fixtures/`.

## Benchmarks

Per-plugin Criterion benches live in `<plugin>/benches/`. Workspace-shared render benches are in `bench-suite/benches/render.rs`. Run with `cargo xtask native bench -p <plugin> --bench <name>` so target-cpu auto-tunes. Standard `cargo clippy --workspace -- -D warnings` does not compile bench/test cfg, so latent `#[cfg(test)]` lints (e.g. `field_reassign_with_default` in some test modules) only surface under `--benches`/`--tests`; the CI gate is the no-flag form.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xxx/tract-plugin-pack](https://github.com/xxx/tract-plugin-pack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
