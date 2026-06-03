---
trigger: always_on
description: `flutter_recorder` is a **Flutter audio recording plugin** that provides low-level audio capture capabilities across all Flutter-supported platforms. It uses the [miniaudio](https://github.com/mackron/miniaudio) C library as the backend and exposes functionality via Dart FFI (Foreign Function Interface).
---

# flutter_recorder - AI Agent Documentation

## Project Overview

`flutter_recorder` is a **Flutter audio recording plugin** that provides low-level audio capture capabilities across all Flutter-supported platforms. It uses the [miniaudio](https://github.com/mackron/miniaudio) C library as the backend and exposes functionality via Dart FFI (Foreign Function Interface).

### Key Features
- Cross-platform audio recording (Android, iOS, Linux, macOS, Windows, Web)
- WAV file recording with pause/resume functionality
- Real-time audio visualization (waveform and FFT data)
- Silence detection with configurable thresholds
- Audio data streaming
- Experimental audio filters (Auto Gain, Echo Cancellation)
- WebAssembly (WASM) support for web platform

## Technology Stack

### Core Technologies
| Component | Technology |
|-----------|------------|
| Language | Dart (SDK ^3.5.1) |
| Native Backend | C++ (C++17) |
| Audio Library | miniaudio (single-header C library) |
| FFI Bindings | `dart:ffi` for IO platforms |
| Web Implementation | WebAssembly via Emscripten |
| Build System | CMake (native), Gradle (Android) |

### Dependencies
```yaml
# Production dependencies
ffi: ^2.1.3
logging: ^1.3.0
meta: ^1.12.0
plugin_platform_interface: ^2.0.2
web: ^1.1.0

# Development dependencies
ffigen: ^13.0.0
flutter_lints: ^4.0.0
very_good_analysis: ^6.0.0
```

## Project Structure

```
flutter_recorder/
├── lib/                          # Dart source code
│   ├── flutter_recorder.dart     # Main library export
│   └── src/
│       ├── flutter_recorder.dart # Main Recorder singleton class
│       ├── enums.dart            # Enums (PCMFormat, RecorderChannels, CaptureErrors)
│       ├── audio_data_container.dart  # Audio data wrapper
│       ├── bindings/             # Platform-specific bindings
│       │   ├── recorder.dart     # Abstract recorder interface
│       │   ├── recorder_io.dart  # FFI implementation for IO platforms
│       │   ├── recorder_web.dart # WASM implementation for web
│       │   ├── js_extension.dart # JavaScript interop helpers
│       │   └── flutter_recorder_bindings_generated.dart  # Auto-generated FFI bindings
│       ├── exceptions/           # Custom exceptions
│       ├── filters/              # Audio filter implementations
│       └── worker/               # Web worker for WASM
│
├── src/                          # C++ native source
│   ├── flutter_recorder.cpp      # Main C++ implementation
│   ├── flutter_recorder.h        # C header for FFI bindings
│   ├── capture.cpp/h             # Audio capture logic
│   ├── analyzer.cpp/h            # FFT analysis
│   ├── common.h                  # Shared definitions
│   ├── enums.h                   # C++ enums
│   ├── circular_buffer.h         # Circular buffer implementation
│   ├── miniaudio.h               # miniaudio library (single header)
│   ├── wav.h                     # WAV file handling
│   ├── fft/                      # FFT implementation (SoLoud)
│   └── filters/                  # C++ audio filters
│       ├── filters.cpp/h
│       ├── autogain.cpp/h
│       ├── echo_cancellation.cpp/h
│       └── generic_filter.h
│
├── web/                          # Web platform assets
│   ├── compile_wasm.sh           # WASM compilation script
│   ├── compile_worker.sh         # Worker compilation script
│   ├── worker.dart               # Web worker Dart code
│   ├── init_recorder_module.dart # WASM module initialization
│   ├── libflutter_recorder_plugin.js    # Generated JS glue
│   └── libflutter_recorder_plugin.wasm  # Generated WASM binary
│
├── example/                      # Example application
│   ├── lib/
│   │   ├── main.dart             # Demo app with UI
│   │   └── ui/                   # Visualization widgets
│   ├── android/                  # Android-specific config
│   ├── ios/                      # iOS-specific config
│   ├── macos/                    # macOS-specific config
│   ├── linux/                    # Linux-specific config
│   └── windows/                  # Windows-specific config
│
├── android/                      # Android plugin config
├── ios/                          # iOS plugin config
├── macos/                        # macOS plugin config
├── linux/                        # Linux plugin config
├── windows/                      # Windows plugin config
│
├── pubspec.yaml                  # Package manifest
├── ffigen.yaml                   # FFI bindings generator config
├── analysis_options.yaml         # Dart linting rules
└── src/CMakeLists.txt            # Native build configuration
```

## Architecture

### Platform Abstraction
The plugin uses a platform abstraction pattern:

1. **Main Interface** (`lib/src/flutter_recorder.dart`): `Recorder` singleton class
2. **Abstract Implementation** (`lib/src/bindings/recorder.dart`): `RecorderImpl` abstract class
3. **IO Implementation** (`lib/src/bindings/recorder_io.dart`): Uses `dart:ffi` to call native code
4. **Web Implementation** (`lib/src/bindings/recorder_web.dart`): Uses WASM and JS interop

### Native Layer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alnitak/flutter_recorder](https://github.com/alnitak/flutter_recorder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
