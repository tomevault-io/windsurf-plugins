---
trigger: always_on
description: Multi-user WebSDR. Streams RF spectrum + audio from RTL-SDR dongles to browsers. Go backend does FFT (shared) + per-client IQ extraction + optional server-side demod/Opus. SolidJS/TypeScript client does demodulation + audio rendering.
---

# AGENTS.md — no-sdr

## What This Is

Multi-user WebSDR. Streams RF spectrum + audio from RTL-SDR dongles to browsers. Go backend does FFT (shared) + per-client IQ extraction + optional server-side demod/Opus. SolidJS/TypeScript client does demodulation + audio rendering.

## Quick Orientation

```
serverng/              → Go backend (chi router, WebSocket, DSP pipeline, dongle management)
  cmd/serverng/        → Entrypoint (main.go)
  internal/api/        → chi REST router + admin endpoints + static file serving
  internal/ws/         → WebSocket manager, binary protocol, backpressure, rate limiting
  internal/dsp/        → FFT, IQ extraction, NCO, Butterworth, decimation, noise blanker, pipeline
  internal/demod/      → Server-side demodulators (FM, AM, SSB, CW, SAM, C-QUAM, RDS)
  internal/codec/      → ADPCM, deflate, Opus (build-tag gated), FFT compression
  internal/dongle/     → Dongle lifecycle: demo/rtl_tcp/rtlsdr(cgo)/airspy/hfp/rsp + Opus pipeline
  internal/config/     → YAML config loader + validation
  internal/history/    → FFT history ring buffer (waterfall backfill)
  internal/gpu/        → GPU offload: VkFFT + Vulkan compute shaders (planned)
shared/src/            → TypeScript types, binary protocol codec, ADPCM codec (zero deps) — merged into client/src/shared/
client/src/shared/     → Protocol constants, codec helpers, types, modes (was separate workspace)
client/src/engine/     → DSP: demodulators, RDS, noise reduction, audio worklet, renderers
client/src/components/ → SolidJS UI: ControlPanel, WaterfallDisplay, FrequencyDisplay
client/src/admin/      → Admin page: AdminPage, admin-store, sections/
client/src/store/      → SolidJS reactive state
client/src/styles/     → Tailwind v4 @theme (design tokens in CSS, no JS config)
config/config.yaml     → Dongle + profile definitions (validated by Go config package)
```

## Architecture (data flow)

```
Dongle (uint8 IQ @ 2.4 MSPS)
  ├─► FftProcessor (1× per dongle, Go) → Float32 dB → codec encode → broadcast all clients
  └─► IqExtractor (1× per client, Go) → NCO + Butterworth + decimate → Int16 sub-band
        ├─► ADPCM/raw → WS → client demod (FM/AM/SSB/CW/SAM/C-QUAM) → audio
        └─► OpusPipeline (Go) → server demod + Opus encode → WS → client decode → audio
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Go 1.23, chi/v5 router, coder/websocket, slog structured logging |
| Frontend | SolidJS, TypeScript, Vite, Tailwind v4 |
| Shared types | TypeScript (npm workspace `shared/`) |
| Build | npm workspaces (shared + client), Go binary (serverng) |
| Dev reload | Air (Go hot reload), Vite HMR (client), concurrently |
| Config | YAML (gopkg.in/yaml.v3), validated by `internal/config` |
| Optional deps | libopus (build tag `opus`), librtlsdr (build tag `rtlsdr`) |

## Key Files (by responsibility)

| File | What it does | Hot path? |
|------|-------------|-----------|
| `serverng/internal/dongle/manager.go` | Dongle lifecycle, per-client IQ pipeline orchestration, FFT broadcast | YES — main data loop (1571 lines) |
| `serverng/internal/dsp/fft_processor.go` | FFT (radix-4), windowing, rate cap, exponential averaging | YES — runs per IQ chunk |
| `serverng/internal/dsp/iq_extractor.go` | Per-client NCO + 4th-order Butterworth + decimation pipeline | YES — O(clients) per chunk |
| `serverng/internal/ws/manager.go` | WS connection lifecycle, broadcast, backpressure, stale-client cleanup | YES — every WS frame |
| `serverng/internal/ws/protocol.go` | Binary WS protocol: pack/unpack, message types, client commands | Every WS message |
| `serverng/internal/demod/fm.go` | Server-side FM demodulator (stereo + RDS) | Per Opus-client |
| `serverng/internal/dongle/opus_pipeline.go` | Server-side demod + Opus encode per client | Per Opus-client |
| `serverng/internal/codec/adpcm.go` | IMA-ADPCM encoder/decoder (4:1 lossy) | Every IQ/FFT frame |
| `serverng/internal/api/router.go` | chi REST routes, WS upgrade, admin API, CORS | Request handler |
| `client/src/engine/sdr-engine.ts` | Client orchestrator (god object) | Dispatches all client work |
| `client/src/engine/demodulators.ts` | All demod classes: FM stereo, AM, C-QUAM, SAM, SSB, CW | Per audio frame |
| `client/src/engine/audio.ts` | AudioWorklet + 5-band EQ + jitter buffer | Per audio frame |
| `shared/src/protocol.ts` | Binary WS protocol: client-side pack/unpack, codec helpers | Every WS message |
| `shared/src/adpcm.ts` | IMA-ADPCM decoder (TypeScript, browser-side) | Every IQ/FFT frame |

## GPU Acceleration (planned)

Offload DSP to GPU via Vulkan + VkFFT for CPU core liberation and higher throughput.

### Operations to Offload

| Operation | GPU Library | Per-Client | Priority |
|-----------|-------------|------------|----------|
| FFT (65536 @ 30fps) | VkFFT | No (shared) | **Critical** |
| Butterworth LPF (4th-order) | Vulkan Compute Shader | Yes | High |
| NCO Frequency Shift | Vulkan Compute Shader | Yes | High |
| Decimation (integer) | Vulkan Compute Shader | Yes | High |
| FM Stereo FIR (2×51-tap) | Vulkan Compute Shader | Yes | **Critical** |

### Hardware Tiers


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gbozo/no-sdr](https://github.com/gbozo/no-sdr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
