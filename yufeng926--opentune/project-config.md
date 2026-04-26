---
trigger: always_on
description: OpenTune is an AI-powered pitch correction standalone application built with the JUCE
---

# AGENTS.md - OpenTune

## Project Overview

OpenTune is an AI-powered pitch correction standalone application built with the JUCE
framework (C++17). It uses NSF-HiFiGAN neural vocoder and RMVPE pitch extraction via
ONNX Runtime for formant-preserving vocal pitch shifting. Company: DAYA. License: AGPL-3.0.

## Build Commands

### Requirements

- CMake 3.22+
- C++17 compiler (MSVC 2022 / Visual Studio 17 on Windows)
- All dependencies are vendored (JUCE, ONNX Runtime 1.17.3, r8brain-free-src)
- `.onnx` model files are tracked via Git LFS -- run `git lfs pull` after cloning

### Configure and Build (Windows)

```bash
mkdir build && cd build
cmake .. -G "Visual Studio 17 2022" -A x64
cmake --build . --config Release
```

Output: `build/OpenTune_artefacts/Release/Standalone/OpenTune.exe`

Post-build automatically copies ONNX Runtime DLLs and model files to the output directory.

### macOS / Linux

Not officially supported yet. The CMakeLists.txt targets Windows/MSVC. A macOS or Linux
build would require providing a platform-appropriate ONNX Runtime library and adjusting
the linker configuration (currently hardcoded to `.lib` / `.dll`).

### LSP Support

`CMAKE_EXPORT_COMPILE_COMMANDS=ON` is set. After cmake configure, `compile_commands.json`
is generated in the build directory for clangd / LSP integration.

## Tests

There is **no test infrastructure**. Tests and CTest registration are commented out in
CMakeLists.txt. No test source files exist in the repository.

## Linting / Formatting

No `.clang-format`, `.clang-tidy`, or `.editorconfig` files are configured. No CI/CD
pipelines exist. Follow the conventions documented below to maintain consistency.

## Project Structure

```
Source/
  PluginProcessor.cpp/h      # Core audio processor (multi-track, playback, mixing)
  Inference/                  # AI model layer (ONNX Runtime wrappers)
    InferenceManager.h        # Singleton, Pimpl pattern
    RMVPEExtractor.h          # F0 pitch extraction
    HifiGANVocoder.h          # Neural vocoder
    ModelFactory.h             # Static factory for model creation
    RenderingManager.h        # Chunk-based render pipeline
    RenderCache.h             # Cached render results
  DSP/                        # Signal processing
    MelSpectrogram.h          # Mel spectrogram computation
    ResamplingManager.h       # Audio resampling (r8brain)
    ScaleInference.h          # Musical scale/key detection
  Services/                   # Background services
    F0ExtractionService.h     # Async F0 extraction with worker threads
  Standalone/                 # Standalone app UI
    PluginEditor.h            # Main editor window (central mediator)
    UI/                       # All UI components
      PianoRollComponent.h    # Piano roll editor
      MainControlPanel.h      # Control panel
      ThemeTokens.h           # Theme system
  Utils/                      # Utilities
    PitchCurve.h              # Immutable-snapshot pitch curve (COW)
    Note.h                    # Note data structures
    AppLogger.h               # Static logging
    UndoAction.h              # Command pattern undo/redo
    LockFreeQueue.h           # Lock-free concurrent containers
  Host/                       # Host integration abstractions
  Audio/                      # Audio subsystem
  Editor/                     # Editor factory
ThirdParty/
  r8brain-free-src-master/    # Vendored resampling library
JUCE-master/                  # Vendored JUCE framework
onnxruntime-win-x64-1.17.3/  # Vendored ONNX Runtime (Windows x64)
models/                       # ONNX model files (Git LFS)
Resources/                    # App icon, fonts
```

## Code Style Guidelines

### Naming Conventions

| Element              | Convention                          | Example                              |
|----------------------|-------------------------------------|--------------------------------------|
| Classes / Structs    | PascalCase                          | `InferenceManager`, `TrackState`     |
| Interfaces (ABC)     | `I` prefix + PascalCase             | `IF0Extractor`, `IVocoder`           |
| Methods              | camelCase                           | `extractF0()`, `prepareToPlay()`     |
| Member variables     | camelCase + trailing underscore     | `sampleRate_`, `inferenceReady_`     |
| Local variables      | camelCase                           | `numSamples`, `clipGain`             |
| Parameters           | camelCase                           | `sampleRate`, `modelDir`             |
| Class constants      | `k` prefix + PascalCase             | `kMaxAudioDurationSec`              |
| Namespace constants  | PascalCase                          | `DefaultSampleRate`                  |
| Legacy constants     | UPPER_SNAKE_CASE                    | `MAX_TRACKS`, `MENU_BAR_HEIGHT`      |
| Enums                | `enum class`, PascalCase type+values| `enum class LogLevel { Info, Error }`|
| Namespaces           | PascalCase                          | `OpenTune`, `AudioConstants`         |
| Template params      | Single letter or PascalCase         | `T`, `ClipType`                      |

### Getters / Setters / Predicates

- Getters: `get` prefix -- `getSnapshot()`, `getPosition()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YuFeng926/OpenTune](https://github.com/YuFeng926/OpenTune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
