---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Go CLI tool for podcast audio preprocessing using embedded FFmpeg. Transforms raw voice recordings into broadcast-ready audio at -16 LUFS through a four-pass adaptive processing pipeline. Uses the [Charm v2 suite](https://charm.land) for the TUI (bubbletea + lipgloss under the `charm.land/<pkg>/v2` import domain).

## Setup commands

- Enter development shell: `nix develop` (or let direnv activate automatically)
- Initialise ffmpeg-statigo submodule and download libraries: `just setup` (fetches latest release tag, updates submodule, runs download-lib automatically)

## Build and test commands

- **Build binary:** `just build` (never use `go build` directly - requires CGO + version injection)
- **Run tests:** `just test`
- **Run linters:** `just lint` (runs `gocyclo`, `ineffassign`, `golangci-lint` including `govet`, and `actionlint`)
- **Clean artifacts:** `just clean`
- **Install to ~/.local/bin:** `just install`
- **VHS demo recording:** `just vhs`

## Architecture

```
cmd/jive-vocals/main.go          # CLI entry, Kong flags, resolveJobs(), ctx + cancel(), starts TUI; runAnalysisOnly() / runAnalysisOnlyWithDeps(); exits 1 when any file fails (cancellation exits 0)
cmd/jive-vocals/pool.go          # runWorkerPool() - bounded concurrent multi-file processing; returns the non-cancellation failure count
cmd/jive-vocals/analysispool.go  # runAnalysisPool() - bounded concurrent multi-file analysis (mirrors pool.go)
internal/
├── audio/reader.go         # FFmpeg demuxer/decoder wrapper (Reader, Metadata, OpenAudioFile)
├── processor/
│   ├── adaptive.go         # AdaptConfig() - derives effective filter settings + diagnostics from Pass 1 measurements; tune steps incl. tuneNoiseReduction() (afftdn enable + measured nf + measured custom band-noise profile via buildAfftdnBandNoise/useCustomAfftdnProfile)
│   ├── adaptive_speech_gate.go  # tuneSpeechGate() - speech-gate threshold/ratio/range/attack/release/knee/detection tuning (voiced-anchored threshold + narrow-gap depth step)
│   ├── advice.go           # GainAdvice() - input-gain advice from input true peak (Clipping/Hot/Quiet/Fine vs -6 dBTP); GainAdviceResult.Message()
│   ├── analyser.go         # AnalyseAudio() - Pass 1: ebur128 + astats + aspectralstats; calls detectVoiceActivity()
│   ├── analyser_vad.go         # detectVoiceActivity() - unified voice-activity detector (histogram + Otsu split + percentile floor, hysteresis runs, adaptive gap-tolerance, spectral veto); elects SpeechProfile + NoiseProfile, sets Noise.VoiceActivated; deriveGateStatistics() - gate-window percentiles (VoicedLowPercentile/NoiseHighPercentile/GateSeparationDB)
│   ├── analyser_candidates_shared.go  # Shared sliding-window refinement (refineToSubregion), interval accumulation, scoreSpeechIntervalWindow, levelVariance
│   ├── analyser_candidates_speech.go  # Speech-candidate scoring (scoreSpeechCandidateGrounded - SNR-primary + saturating duration adequacy + consistency tie-break) and election (findBestSpeechRegion, highest-score)
│   ├── analyser_noise_seed.go  # Pre-scan noise-floor seed estimators (Noise.FloorPrescan, anchors the VAD split clamp) + golden-window bounds
│   ├── analyser_bands.go       # Region-scoped sibilant/body band RMS for de-esser intensity; runs its 2 band decodes as bounded goroutines via runBandMeasurements
│   ├── analyser_noise_bands.go # measureNoiseBands() - Pass 1 15-band RMS spectrum of the elected room-tone region (band centres 80 Hz to 24 kHz); skips decodes when custom afftdn is ineligible; sets NoiseProfile.BandNoise + BandsMeasured for afftdn's custom noise profile; runs its 15 band decodes as bounded goroutines via runBandMeasurements
│   ├── analyser_band_runner.go # runBandMeasurements() - shared bounded-goroutine runner for the post-loop band decodes; bandMeasureSem package semaphore (buffered channel sized runtime.NumCPU(), shared across all files and both speech/noise band groups); bandProgressTracker + pure bandPhaseProgress progress helpers
│   ├── analyser_metrics.go     # IntervalSample, SpectralMetrics, per-250ms metric accumulation
│   ├── analyser_output.go      # MeasureOutputRegions() - before/after region comparison
│   ├── encoder.go          # Output file encoder wrapper
│   ├── filters.go          # BaseFilterConfig, EffectiveFilterConfig, AdaptiveDiagnostics, filter builders, BuildFilterSpec(), DefaultFilterConfig()
│   ├── frame_processor.go  # runFilterGraph(), FrameLoopConfig - shared filter graph execution
│   ├── limiter.go          # deriveLimiterAndPreGain() + limiter/pre-gain derivation, buildPreLimiterPrefix()/buildBrickwallLimiter(), limiterPlan, LimiterDiagnostics; holds the limiter tuning constants (minLimiterCeilingDB, brickwallTruePeakHeadroomDB, measurementCushionDB, linearSafetyMargin)
│   ├── normalise.go        # ApplyNormalisation() - Pass 3/4: loudnorm measurement + application; holds the loudnormTP bounds
│   ├── processor.go        # ProcessAudio(), AnalyseOnlyDetailed() - pass orchestration
│   ├── quality.go          # ComputeQualityScore() - Processed star rating: output vs -16 LUFS spec (saturates at 5★)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linuxmatters/jive-vocals](https://github.com/linuxmatters/jive-vocals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
