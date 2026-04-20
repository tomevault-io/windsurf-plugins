---
trigger: always_on
description: AI speech models for Apple Silicon (MLX Swift). ASR, TTS, speech-to-speech, VAD, diarization, speech enhancement.
---

# Agent Instructions

AI speech models for Apple Silicon (MLX Swift). ASR, TTS, speech-to-speech, VAD, diarization, speech enhancement.

## Workflow

- **Never commit, push, or comment on GitHub without explicit user confirmation.** Draft first, ask to confirm, then execute.
- **Every README.md change must update all 9 translations** (`README_zh.md`, `README_ja.md`, `README_ko.md`, `README_es.md`, `README_de.md`, `README_fr.md`, `README_hi.md`, `README_pt.md`, `README_ru.md`). No exceptions.

## Git Conventions

- Never mention Claude, AI, or any AI tool in commit messages, PR descriptions, or co-author tags
- No `Co-Authored-By` lines in commits
- **Never amend commits or force push** unless the user explicitly asks for it
- Always use branches and PRs — commit history must be preserved

## Build

```bash
# Release build (recommended)
make build

# Debug build
make debug

# Run tests (builds debug first)
make test

# Clean
make clean
```

Or manually:

```bash
swift build -c release --disable-sandbox
./scripts/build_mlx_metallib.sh release
```

The metallib step compiles MLX Metal shaders — without it, inference runs ~5x slower due to JIT shader compilation.

## Skills (Slash Commands)

Project skills in `.claude/skills/`:

| Command | Description |
|---------|-------------|
| `/build` or `/build release` | Release build with metallib |
| `/build debug` | Debug build |
| `/test` or `/test unit` | Run unit tests (skip E2E) |
| `/test e2e` | Full test suite with model downloads |
| `/test FilterName` | Run specific test filter |
| `/benchmark asr` | Benchmark ASR speed |
| `/benchmark tts` | Benchmark TTS speed |
| `/benchmark vad` | VAD benchmark on VoxConverse |
| `/benchmark diarize` | DER benchmark on VoxConverse |

## Project Structure

- `Sources/Qwen3ASR/` — Speech-to-text (Qwen3-ASR)
- `Sources/ParakeetASR/` — Speech-to-text (Parakeet TDT, CoreML)
- `Sources/ParakeetStreamingASR/` — Streaming speech-to-text (Parakeet EOU 120M, CoreML)
- `Sources/OmnilingualASR/` — Speech-to-text (Meta wav2vec2 CTC, 1,672 languages, CoreML 300M + MLX 300M/1B/3B/7B)
- `Sources/Qwen3TTS/` — Text-to-speech (Qwen3-TTS)
- `Sources/CosyVoiceTTS/` — Text-to-speech (CosyVoice3, streaming)
- `Sources/KokoroTTS/` — Text-to-speech (Kokoro-82M, CoreML, iOS-ready)
- `Sources/Qwen3TTSCoreML/` — Text-to-speech (Qwen3-TTS 0.6B, CoreML, 6-model pipeline)
- `Sources/PersonaPlex/` — Speech-to-speech (PersonaPlex 7B, full-duplex)
- `Sources/SpeechVAD/` — VAD (Silero + Pyannote), speaker diarization, speaker embedding (WeSpeaker)
- `Sources/SpeechEnhancement/` — Noise suppression (DeepFilterNet3, CoreML)
- `Sources/Qwen3Chat/` — On-device LLM chat (Qwen3.5-0.8B, MLX + CoreML, INT4/INT8)
- `Sources/MLXCommon/` — Shared MLX utilities (weight loading, quantized layers, memory estimation, `SDPA` multi-head attention helper)
- `Sources/AudioCommon/` — Audio I/O, protocols, HuggingFace downloader, shared `SentencePieceModel` protobuf reader
- `Sources/AudioCLILib/` — CLI commands
- `Sources/AudioCLI/` — CLI entry point (`audio` binary)
- `Tests/` — Unit and integration tests
- `scripts/` — Model conversion (PyTorch → MLX/CoreML), benchmarking
- `Examples/` — Demo apps (PersonaPlexDemo, SpeechDemo, iOSEchoDemo)

## Key Conventions

- Swift 6, macOS 15+ / iOS 18+, Apple Silicon (M-series)
- MLX for GPU inference (Metal), CoreML for Neural Engine (DeepFilterNet3, Kokoro, Qwen3-TTS, Silero VAD optional)
- Models are downloaded from HuggingFace on first use, cached in `~/Library/Caches/qwen3-speech/`
- All audio processing uses Float32 PCM, resampled to model-specific rates internally
- `DiarizedSegment`, `SpeechSegment`, protocol types defined in `Sources/AudioCommon/Protocols.swift`
- Tests that use MLX arrays require the compiled metallib; config/logic-only tests work without it

## Testing

Safe tests (no GPU/model download required):
```bash
make test
```

Full test suite (requires metallib + model downloads):
```bash
make test
```

### Testing requirements for new code

**Every new feature, model, or module MUST include tests:**

- **Unit tests**: Config parsing, data structures, weight loading, math/DSP logic — no GPU or model downloads needed
- **E2E tests**: Full pipeline with real model weights — verify correct output (e.g., ASR round-trip, correct transcription text)
- **Regression tests**: When fixing bugs, add a test that would have caught the bug

**Test organization**: Place tests in `Tests/<ModuleName>Tests/`. Follow existing patterns (e.g., `Qwen3ASRTests/`, `SpeechVADTests/`).

**E2E test naming**: Prefix E2E test classes with `E2E` (e.g., `E2ETranscriptionTests`, `E2EDiarizationTests`). CI uses `--skip E2E` regex to filter out all E2E tests that require model downloads — only unit tests run in the pipeline. E2E tests run locally with `make test` (full suite). **CRITICAL**: Any test class that downloads models or requires GPU inference MUST be prefixed with `E2E`. Unit test classes must NOT contain `E2E` in their name.

**What to test per category:**
| Change | Required tests |
|--------|---------------|
| New model/module | Unit (config, weight loading) + E2E (inference produces correct output) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soniqo/speech-swift](https://github.com/soniqo/speech-swift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
