---
trigger: always_on
description: macOS menu bar voice input tool with dual-engine local ASR, multi-provider cloud ASR, and LLM post-processing.
---

# Type4Me — Development Guide

## Overview

macOS menu bar voice input tool with dual-engine local ASR, multi-provider cloud ASR, and LLM post-processing.
Local ASR: SenseVoice via native sherpa-onnx (streaming) + Qwen3-ASR (final calibration, Python WebSocket service managed by `SenseVoiceServerManager`).
Cloud ASR: 8 providers implemented (Volcano, OpenAI, Deepgram, AssemblyAI, ElevenLabs, Soniox, Bailian, Baidu).
Swift Package Manager project, no Xcode project file. Optional `sherpa-onnx.xcframework` for punctuation restoration.

## Build & Run

```bash
# Qwen3-ASR server setup (optional, Apple Silicon only)
cd qwen3-asr-server && python3.12 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt && cd ..

# Optional: punctuation restoration module (~5 min, requires cmake)
bash scripts/build-sherpa.sh

swift build -c release
```

The built binary is at `.build/release/Type4Me`. To package it as a `.app` bundle, see `scripts/deploy.sh`.

## Build Variants

Three product variants built from the same codebase via conditional compilation flags:

| Variant | `HAS_SHERPA_ONNX` | `HAS_CLOUD_SUBSCRIPTION` | Arch | Description |
|---------|---|---|---|---|
| **pure** | no | no | universal | Open-source cloud edition (BYOK API keys) |
| **official** | no | yes | universal | Official member edition (subscription + cloud proxy) |
| **local** | yes | no | arm64 | Open-source local edition (bundled SenseVoice + Qwen3-ASR) |

### How it works

- `HAS_SHERPA_ONNX`: controlled by `Frameworks/sherpa-onnx.xcframework/Info.plist` presence (existing pattern)
- `HAS_CLOUD_SUBSCRIPTION`: controlled by `Type4Me/CloudSubscription/marker` file presence
- `Package.swift` detects these files at manifest evaluation time and sets compiler defines + source excludes
- `build-dmg.sh` temporarily hides marker files to build each variant

### Build commands

```bash
# Open-source cloud edition (no subscription, no local ASR)
VARIANT=pure bash scripts/build-dmg.sh

# Official member edition (with subscription system)
VARIANT=official bash scripts/build-dmg.sh

# Open-source local edition (bundled models, Apple Silicon only)
VARIANT=local bash scripts/build-dmg.sh
```

### Subscription code location

All subscription/cloud-proxy code lives in `Type4Me/CloudSubscription/` (13 files). Main code uses `#if HAS_CLOUD_SUBSCRIPTION` guards at ~11 touch points. When the marker is absent, the directory is excluded from compilation entirely.

**Important**: SPM caches manifest evaluation in `~/Library/Caches/org.swift.swiftpm`. When switching variants manually (not via build-dmg.sh), clear this cache: `rm -rf .build ~/Library/Caches/org.swift.swiftpm`

## ASR Provider Architecture

Multi-provider ASR support via `ASRProvider` enum + `ASRProviderConfig` protocol + `ASRProviderRegistry`.

- `ASRProvider` enum: 15 cases + conditional `cloud` case (sherpa/openai/azure/google/aws/deepgram/assemblyai/elevenlabs/volcano/aliyun/bailian/tencent/baidu/iflytek/custom, plus `cloud` when `HAS_CLOUD_SUBSCRIPTION`)
- Each provider has its own Config type (e.g., `SherpaASRConfig`, `VolcanoASRConfig`) defining `credentialFields` for dynamic UI rendering
- `ASRProviderRegistry`: maps provider to config type + client factory; `capabilities` indicates availability and streaming support
- **Fully implemented**: sherpa (local, batch), volcano (streaming), deepgram (streaming), assemblyai (streaming), elevenlabs (streaming), soniox (streaming), bailian (streaming), baidu (streaming), openai (batch)
- **Config only (no client)**: azure, google, aws, aliyun, tencent, iflytek, custom

### Adding a New Provider

1. Create a Config file in `Type4Me/ASR/Providers/`, implementing `ASRProviderConfig`
2. Write the client (implementing `SpeechRecognizer` protocol)
3. Register `createClient` in `ASRProviderRegistry.all`

## Local ASR Architecture (SenseVoice + Qwen3-ASR)

### Dual-Engine Design
- **SenseVoice**: Native sherpa-onnx integration (Swift), provides real-time streaming recognition (partial results as you speak). No Python dependency.
- **Qwen3-ASR** (`qwen3-asr-server/`): Python WebSocket service using MLX (Metal GPU), provides final calibration on complete audio for higher accuracy. Apple Silicon only.
- `SenseVoiceServerManager`: manages the Qwen3-ASR Python server process, auto-detects Apple Silicon vs Intel, assigns dynamic ports, saves PIDs for graceful shutdown

### Recognition Pipeline
1. `SenseVoiceWSClient` connects to local Python servers via WebSocket
2. Three modes: SenseVoice streaming only, Qwen3-only (final result), or hybrid (SenseVoice streaming + Qwen3 final calibration)
3. Qwen3 incremental speculative transcription with debounce for progressive results
4. `SherpaPunctuationProcessor` (optional) — CT-Transformer post-processing adds punctuation (requires `sherpa-onnx.xcframework`)

### Models
- One streaming model in `ModelManager.StreamingModel`: `senseVoiceSmall` (~228MB, zh/en/yue/ja/ko)
- Auxiliary models: `offlineParaformer` (~700MB), `punctuation` CT-Transformer (~72MB)
- Models downloaded from GitHub releases (tar.bz2), stored at `~/Library/Application Support/Type4Me/models/`

### SherpaOnnx Integration (optional, for punctuation only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joewongjc/type4me](https://github.com/joewongjc/type4me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
