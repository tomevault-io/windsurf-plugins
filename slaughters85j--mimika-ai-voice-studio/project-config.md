---
trigger: always_on
description: Single-shot context for any Codex session working in this repo. Read first.
---

# AGENTS.md

Single-shot context for any Codex session working in this repo. Read first.

> **Note:** This file is intentionally checked in (no symlink trick) so the project is self-contained for fresh sessions.

---

## Project Overview

**mimika-ai-voice-studio** is a native Swift / SwiftUI macOS app that replaces the existing Electron-based pocket-tts frontend with a fully on-device, Python-free TTS application. It runs the Kyutai pocket-tts model end-to-end via Core ML `.mlpackage` artifacts (CaLM + Mimi codec), with no Python server, no PyInstaller bundle, and no network dependency for synthesis.


---

## Architecture (Core ML pipeline)

```
At app launch (once per voice selection):
  voice_kv_states/<voice>.safetensors  →  load fp16 K/V tensors
                                       →  MLState.write_state into all 4 mlpackage states
                                          (prompt_phase + calm_stateful share state contents
                                           by re-using the same KV layout; mimi_stateful has
                                           its own separate per-frame state)

Per synthesis call:
  User text → SentencePiece (Swift) → token IDs (padded to T_TEXT_MAX=128)
                                         ↓
        prompt_phase.mlpackage(text_tokens, voice_offset=T_voice, text_length=N)
                                         ↓
                       state buffers now contain voice KV (pos 0..T_voice)
                       + text KV (pos T_voice..T_voice+N)
                                         ↓
                       returns t_prompt = T_voice + N
                                         ↓
        ┌──────── per-frame autoregressive loop (frame_idx = 0, 1, 2, ...) ────────┐
        │                                                                           │
        │  calm_stateful.mlpackage(prev_latent, offset=t_prompt + frame_idx, noise) │
        │                          ──► one latent frame, EOS flag                   │
        │                          (KV state mutated in-place at the offset slot)   │
        │                                       │                                   │
        │                                       ▼                                   │
        │  mimi_stateful.mlpackage(latent)  ──► 1920 PCM samples (80 ms @ 24 kHz)  │
        │                                       │                                   │
        └───────────────────────────────────────┼───────────────────────────────────┘
                                                ▼
                                       AsyncStream<PCMFrame>
                                                ↓
                                       StreamingPlayer (AVAudioEngine)
                                                ↓
                                  speakers + WAV/AAC/MP3 encoder
```

**State-sharing note:** `prompt_phase` and `calm_stateful` were converted with **identical state-buffer shapes and names** (12 buffers: `kv_k_0..5`, `kv_v_0..5`, each `[1, 512, 16, 64]` fp16). Swift maintains ONE logical KV cache and writes it into both models' state objects. The first call (`prompt_phase`) populates positions `0..t_prompt`; subsequent calls (`calm_stateful`) extend it one slot per frame.

- **Frame rate:** 12.5 Hz (80 ms / frame)
- **Sample rate:** 24 kHz mono
- **Steady-state throughput:** ~38 fps on M1 Ultra (~3× real-time)
- **EOS:** CaLM's EOS head signals end; pipeline runs `frames_after_eos` more then stops
- **Numerical equivalence:** validated end-to-end vs PyTorch reference; e2e spectrum correlation 0.97

Conversion details are intentionally kept out of the app repo. The app consumes
checked-in Core ML resources and should not depend on a separate conversion
workspace at runtime.

---

## Project Layout

```
mimika-ai-voice-studio/
├── CLAUDE.md
├── AGENTS.md                          ← this file
├── mimika-ai-voice-studio.xcodeproj/
├── mimika-ai-voice-studio/
│   ├── pocket_tts_macosApp.swift     (@main entry point)
│   ├── ContentView.swift             (NavigationSplitView; routes .needsModelDownload → FirstLaunchSetupView)
│   ├── road-map.md
│   ├── App/
│   │   ├── AppState.swift            (global app state + engine ownership + .needsModelDownload gate)
│   │   └── SynthesisStatus.swift
│   ├── Models/
│   │   ├── BundledVoice.swift        (stock voice catalog entry)
│   │   └── ChatModels.swift
│   ├── Engine/
│   │   ├── TTS/
│   │   │   ├── TTSEngine.swift           (Core ML synthesis orchestrator)
│   │   │   ├── TTSEngineProtocol.swift   (testable engine surface)
│   │   │   ├── Tokenizer.swift           (SentencePiece wrapper)
│   │   │   ├── SentencePieceTokenizer.swift
│   │   │   ├── VoiceLoader.swift         (safetensors → MLMultiArray)
│   │   │   ├── VoiceManager.swift        (saved-voices/ catalog + import + orphan recovery)
│   │   │   ├── ModelPaths.swift          (dual-source resolution: downloaded > bundle > throw)
│   │   │   ├── BundledMLModel.swift      (4-case catalog: URL + SHA + display strings)
│   │   │   ├── BundledMLModelManager.swift (@MainActor @Observable; download → verify → compile → install)
│   │   │   ├── BundledMLModelManagerTypes.swift (DownloadState + ManagerError)
│   │   │   ├── FishEngine.swift

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slaughters85j/mimika-ai-voice-studio](https://github.com/slaughters85j/mimika-ai-voice-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
