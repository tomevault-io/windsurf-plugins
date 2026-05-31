---
trigger: always_on
description: transforms:
---

# noisekit — Claude Instructions

**Always update this file when making notable changes** (new commands, new presets, architectural decisions, scoring changes, dependency additions).

**Always update README.md** when changing presets, CLI flags, output format, or any user-facing behavior.

## Project

`noisekit` is a `uvx`-compatible Python CLI that generates degraded speech datasets from clean HuggingFace corpora. It simulates six atomic audio degradation scenarios — telecom (G.711 calls), low-bitrate codec compression, noisy environments (real ambient noise), far-field reverb, and clipping distortion — plus compound multi-condition scenarios built by chaining atomic presets. Designed for ASR noise-robustness benchmarking. A `clean_reference` control completes the catalog.

## Package Management

Use **UV** for everything: `uv add`, `uv run`, `uv sync`. Never use pip directly.

Key runtime dependencies: `audiomentations>=0.38`, `lameenc>=1.4` (pure-Python MP3 encoder used by `Mp3Compression` in `telecom` and `low_bitrate`; no system ffmpeg needed), `torchmetrics>=1.7.0` (NISQA scoring — downloads ~50 MB model weights to `~/.torchmetrics/NISQA/` on first use), `pyroomacoustics` (room acoustics simulation for `reverb` — now a core dependency, no extra install needed).

## Architecture

```
noisekit/
├── cli.py          # Typer app — 3 commands: generate, score, list-presets
├── pipeline.py     # generate + score logic
├── dataset.py      # HuggingFace dataset loading (soundfile decoder, no torchcodec)
├── transforms.py   # Preset loading; returns PresetTransforms(full, scoring, scoring_sr)
├── scoring.py      # PESQ + SNR + NISQA; PESQ NB at 8 kHz for telephony presets
├── noise_cache.py  # Auto-downloads MUSAN music+noise for noise
└── presets/        # YAML preset files bundled with the package

```

## CLI

```bash
noisekit generate --dataset <hf-name> --samples N --presets P1 P2 --output ./out --seed 42
noisekit generate ... --presets noise --noise-dir /path/to/noise_wavs
noisekit generate ... --no-nisqa          # skip NISQA (no model download, faster)
noisekit score ./audio_dir [--reference-dir ./ref] [--output scores.json]
noisekit score ./audio_dir --no-nisqa     # skip NISQA for standalone scoring
noisekit list-presets [--verbose]
```

Custom presets: `--preset-file ./my_preset.yaml`

The `noise` preset uses a directory of background-noise WAVs. If `--noise-dir` is omitted, noisekit auto-downloads a small MUSAN **noise-only** subset (~20 files, ~120 MB) from `Aynursusuz/musan-audio-dataset` on HuggingFace to `~/.cache/noisekit/noise/musan_ambient/` on first use. Both `speech` and `music` classes are excluded: speech pollutes ASR/PESQ scoring; music sounds artificial as a background and is indistinguishable from white noise at low levels. Only label 2 (`noise` — wind, rain, traffic, machinery) is downloaded.

Pass `--noise-dir /path/to/wavs` to use your own corpus (e.g. MUSAN, DEMAND, FSD50K) instead. Inside a preset YAML, use the literal string `${NOISE_DIR}` as a parameter value and `transforms.load_preset` substitutes the resolved path at load time. Auto-download is wired in `pipeline.run_generate` via `noise_cache.ensure_default_noise_dir()`, gated by `transforms.preset_requires_noise_dir()`.

### MUSAN download — shard strategy

`Aynursusuz/musan-audio-dataset` is **sorted by label**: speech fills parquet shards 0–21, music+noise occupy shards 22–44 (music-first within that range, then noise). `noise_cache.py` bypasses speech entirely by loading only shards 22–44 via `hf://` URLs, then filters to `label == 2` (noise only). The shard list is shuffled before streaming so noise-heavy shards are hit early; a `buffer_size=200` shuffle adds within-shard diversity. Constants `_N_SHARDS = 45` and `_FIRST_AMBIENT_SHARD = 22` must be updated if the dataset is re-sharded. If the download yields zero noise samples, bisect by testing individual shards to find where the noise class begins.

## Preset YAML Format

```yaml
name: my_preset
description: "..."
transforms:
  - type: <audiomentations class>
    parameters:
      key: value
    p: 1.0
```

Built-in presets:

### Atomic Presets

| Preset                 | Scenario                                              | Bandwidth           | PESQ mode | Target MOS |
| ---------------------- | ----------------------------------------------------- | ------------------- | --------- | ---------- |
| `clean_reference`      | Minimal gain normalization (PESQ ceiling)             | full                | WB 16 kHz | 4.0-4.5    |
| `telecom`              | G.711 call + low-bitrate MP3 codec artifacts          | 300-3400 Hz @ 8 kHz | NB 8 kHz  | 2.0-3.5    |
| `low_bitrate`    | Wideband low-bitrate MP3 compression (16-32 kbps)     | 80-7500 Hz @ 16 kHz | WB 16 kHz | 1.5-2.5    |
| `noise`                | Real ambient noise via `AddBackgroundNoise`           | up to 8-12 kHz      | WB 16 kHz | 2.0-3.5    |
| `clipping`             | Microphone overload / ADC saturation (`ClippingDistortion` 10-25%) | full | WB 16 kHz | 2.0-3.5    |
| `reverb`               | Far-field reverberant room via `RoomSimulator`                              | full | WB 16 kHz | 2.0-3.5 |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [karamouche/noisekit](https://github.com/karamouche/noisekit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
