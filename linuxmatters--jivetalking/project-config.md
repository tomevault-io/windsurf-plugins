---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Go CLI tool for podcast audio preprocessing using embedded FFmpeg. Transforms raw voice recordings into broadcast-ready audio at -16 LUFS through a four-pass adaptive processing pipeline. Uses [Charm Bubbletea](https://github.com/charmbracelet/bubbletea) for the TUI.

## Setup commands

- Enter development shell: `nix develop` (or let direnv activate automatically)
- Initialise ffmpeg-statigo submodule and download libraries: `just setup` (fetches latest release tag, updates submodule, runs download-lib automatically)

## Build and test commands

- **Build binary:** `just build` (never use `go build` directly - requires CGO + version injection)
- **Run tests:** `just test`
- **Run linters:** `just lint` (runs `go vet`, `gocyclo`, `ineffassign`, `golangci-lint`, `actionlint`)
- **Clean artifacts:** `just clean`
- **Install to ~/.local/bin:** `just install`
- **VHS demo recording:** `just vhs`

## Architecture

```
cmd/jivetalking/main.go     # CLI entry, Kong flags, starts TUI + processing goroutine
internal/
├── audio/reader.go         # FFmpeg demuxer/decoder wrapper (Reader, Metadata, OpenAudioFile)
├── processor/
│   ├── adaptive.go         # AdaptConfig() - tunes FilterChainConfig from Pass 1 measurements
│   ├── analyzer.go         # AnalyzeAudio() - Pass 1: ebur128 + astats + aspectralstats; silence/speech detection
│   ├── analyzer_candidates.go  # Silence candidate scoring and election
│   ├── analyzer_metrics.go     # IntervalSample, SpectralMetrics, per-250ms metric accumulation
│   ├── analyzer_output.go      # MeasureOutputRegions() - before/after region comparison
│   ├── encoder.go          # Output file encoder wrapper
│   ├── filters.go          # FilterChainConfig, filter builder funcs, BuildFilterSpec(), DefaultFilterConfig()
│   ├── frame_processor.go  # runFilterGraph(), FrameLoopConfig - shared filter graph execution
│   ├── normalise.go        # ApplyNormalisation() - Pass 3/4: loudnorm measurement + application
│   └── processor.go        # ProcessAudio(), AnalyzeOnly() - pass orchestration
├── logging/
│   ├── analysis_display.go # DisplayAnalysisResults() - console output for --analysis-only mode
│   ├── recording_tips.go   # Actionable recording advice from measurements
│   ├── report.go           # GenerateReport() - detailed analysis log files (--logs)
│   └── table.go            # MetricRow, reusable multi-column table formatting (Input/Filtered/Final)
├── ui/
│   ├── analysis_model.go   # AnalysisModel - Bubbletea model for --analysis-only progress TUI
│   ├── messages.go         # ProgressMsg, FileStartMsg, FileCompleteMsg, AllCompleteMsg
│   ├── model.go            # Main processing TUI model
│   └── views.go            # TUI rendering
└── cli/                    # Help styling, version output, error formatting
```

**Data flow (processing):** `main.go` spawns goroutine → `ProcessAudio()` → Pass 1 (`AnalyzeAudio`) → `AdaptConfig()` → Pass 2 (filter chain) → Pass 3/4 (`ApplyNormalisation`) → sends `ui.*Msg` to TUI via `tea.Program.Send()`.

**Data flow (analysis-only):** `main.go` → `runAnalysisOnly()` → `AnalyzeOnly()` → Pass 1 + `AdaptConfig()` → `AnalysisModel` TUI shows progress → `DisplayAnalysisResults()` prints report to console. No output files created.

## Audio processing pipeline

**Four-pass architecture:**

1. **Pass 1 (Analysis):** Measures LUFS, true peak, LRA, noise floor, spectral characteristics; detects silence/speech regions via 250ms interval sampling
2. **Pass 2 (Processing):** Applies adaptive filter chain tuned to measurements; output measured for before/after comparison
3. **Pass 3 (Measuring):** Optionally prepends `volume` (pre-gain) + `alimiter` (Volumax) when limiting is active, then runs loudnorm in measurement mode (JSON output via FFmpeg log capture) to get input stats for linear mode; measures the post-limiter signal so `measured_I`/`measured_TP` are accurate
4. **Pass 4 (Normalising):** Applies `volume` (pre-gain, when ceiling clamped) + `alimiter` (Volumax) + `loudnorm` (linear mode) + `adeclick`; pre-gain raises very quiet recordings so the alimiter can use a viable ceiling; `alimiter` creates headroom so loudnorm achieves full linear gain to reach -16 LUFS

**Filter chain order (Pass 2):**
```
downmix → ds201_highpass → ds201_lowpass → noiseremove (anlmdn+compand) → ds201_gate → la2a_compressor → deesser → analysis → resample
```

Order rationale: downmix to mono first; HP/LP removes frequency extremes before gate (DS201 frequency-conscious side-chain pattern); denoising before gating (lowers noise floor for gate); compression before de-essing (compression emphasises sibilance); analysis measures processed signal; resample standardises output format last.

**Normalisation (Pass 3/4):**
```
Pass 3: [volume (pre-gain, when clamped) → alimiter (Volumax)] → loudnorm (measure-only, print_format=json) → captures LoudnormStats JSON
Pass 4: volume (pre-gain, when clamped) → alimiter (Volumax, peak reduction) → loudnorm (linear mode, input stats from Pass 3) → adeclick
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linuxmatters/jivetalking](https://github.com/linuxmatters/jivetalking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
