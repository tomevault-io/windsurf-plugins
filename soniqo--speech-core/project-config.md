---
trigger: always_on
description: speech-core — voice agent pipeline engine in C++17. Provides:
---

# Agent Instructions

## Project

speech-core — voice agent pipeline engine in C++17. Provides:

1. **Orchestration core** — state machine, turn detection, interruption handling, speech queue, conversation context, streaming VAD state machine, audio utilities. Zero ML dependencies, pure C++17.
2. **Abstract interfaces** — `STTInterface`, `TTSInterface`, `VADInterface`, `TurnCompletionInterface`, `EnhancerInterface`, `EchoCancellerInterface`, `LLMInterface`, plus the batch diarization interfaces `SegmentationInterface`, `EmbeddingInterface`, `DiarizerInterface` (with `SegmentationWindow` / `DiarizedSegment` / `DiarizerConfig` types) in `include/speech_core/interfaces.h`. Consumers (speech-swift, speech-android, speech-cloud, …) implement these with their own model backends.
3. **Optional ONNX reference implementations** — `SileroVad`, `ParakeetStt`, `KokoroTts`, `DeepFilterEnhancer`, `OnnxSmartTurn` in `include/speech_core/models/`. Compiled in only when `SPEECH_CORE_WITH_ONNX=ON`.
4. **Optional LiteRT reference implementations** — `LiteRTSileroVad`, `LiteRTParakeetStt`, `LiteRTKokoroTts`, `LiteRTVoxCPM2Tts`, `LiteRTChatterboxTts`, `LiteRTWeSpeakerEmbedding`, `LiteRTPyannoteSegmentation`, `LiteRTOmnilingualStt`, `LiteRTNemotronStreamingStt` in `include/speech_core/models/`. Compiled in only when `SPEECH_CORE_WITH_LITERT=ON`. Backed by `libLiteRt` from Google's `ai-edge-litert` package (extracted from the PyPI wheel by `scripts/fetch_litert.sh`). DeepFilter LiteRT does not exist yet. `LiteRTKokoroTts` ships the validated three-stage 60-frame FP32 bundle, enforces a 56-frame right-context guard, and is enabled on Windows and Android where the validated runtimes export the required TensorFlow Lite interpreter C API. `LiteRTChatterboxTts` (multilingual TTS, `soniqo/Chatterbox-LiteRT`, C ABI in `chatterbox_c.h`) defaults to greedy decoding; ship the fp16 T3 bundle for Arabic (int8 is English-quality only).
5. **Diarization** — `DiarizationPipeline` (`DiarizerInterface`) in `include/speech_core/diarization/`. Pure C++17, no ML-runtime dependency, built into the **core** library; composes a `SegmentationInterface` + `EmbeddingInterface` with constrained agglomerative clustering.

## Structure

- `include/speech_core/` — public headers (`pipeline/`, `vad/`, `audio/`, `tools/`, `models/`, `util/`, `interfaces.h`, `speech_core_c.h`)
- `src/` — implementations matching the header layout
- `tests/` — unit and integration tests (`test_*.cpp`, picked up via `file(GLOB)`)
- `docs/` — `pipeline.md`, `interfaces.md`, `models.md`, `c-api.md`, `tools.md`
- `scripts/` — utility scripts

Sibling repos under `~/repos/`:

- **speech-android** — Android SDK + JNI bridge to speech-core
- **speech-swift** — Swift Package consuming speech-core as a prebuilt xcframework binary target; CoreML/MLX models implement the same conceptual interfaces (see speech-swift's `docs/shared-protocols.md`)
- **speech-models** — model artifacts on HuggingFace: ONNX + LiteRT under `soniqo/`, CoreML/MLX (Apple) under `aufklarer/`
- **speech-cloud** — server-side counterpart

## Build

### Default (orchestration only, no ML deps)

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
cd build && ctest
```

### With ONNX Runtime reference models

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release \
    -DSPEECH_CORE_WITH_ONNX=ON \
    -DORT_DIR=/path/to/onnxruntime
cmake --build build
```

`ORT_DIR` must contain `include/onnxruntime_c_api.h` and a platform shared library:
- macOS: `lib/libonnxruntime.dylib`
- Linux: `lib/libonnxruntime.so`
- Android: `lib/${ANDROID_ABI}/libonnxruntime.so`

### With LiteRT reference models

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release \
    -DSPEECH_CORE_WITH_LITERT=ON \
    -DLITERT_DIR=/path/to/litert
cmake --build build
```

`LITERT_DIR` must contain `libLiteRt.{so,dylib,dll}` (extracted from Google's `ai-edge-litert` PyPI wheel by `scripts/fetch_litert.sh`). Headers are vendored in `third_party/litert/`; no separate include path needed. Both `SPEECH_CORE_WITH_ONNX` and `SPEECH_CORE_WITH_LITERT` are independent and can be enabled together.

## CMake targets

- **`speech_core`** — static library, orchestration + interfaces + audio utilities. No ORT. Always built.
- **`speech_core_models`** — static library, ONNX Runtime reference implementations. Links `speech_core` + imported `onnxruntime`. Only built when `SPEECH_CORE_WITH_ONNX=ON`.
- **`speech_core_models_litert`** — static library, LiteRT reference implementations (Silero VAD, Parakeet STT, Kokoro TTS, VoxCPM2 TTS, WeSpeaker embedding, Pyannote segmentation, Omnilingual STT, Nemotron streaming STT, VoxCPM2 tokenizer). Links `speech_core` + imported `litert` (`libLiteRt` from ai-edge-litert). Only built when `SPEECH_CORE_WITH_LITERT=ON`. (`DiarizationPipeline` is pure C++ and lives in `speech_core` itself, not here.)

Consumers link the targets they need:

```cmake
target_link_libraries(my_app PRIVATE speech_core)                         # orchestration only
target_link_libraries(my_app PRIVATE speech_core speech_core_models)       # + ONNX models
target_link_libraries(my_app PRIVATE speech_core speech_core_models_litert) # + LiteRT models
```

## Key files

| File | Purpose |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soniqo/speech-core](https://github.com/soniqo/speech-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
