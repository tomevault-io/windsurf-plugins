---
trigger: always_on
description: This file provides guidelines for Claude when working with the Amplitude Audio SDK codebase.
---

# Claude Configuration for Amplitude Audio SDK

This file provides guidelines for Claude when working with the Amplitude Audio SDK codebase.

## Project Overview

Amplitude is a full-featured and cross-platform audio engine designed for games. It provides efficient audio mixing, spatial audio (HRTF, Ambisonic), attenuation models, bus management, and more through a data-driven configuration approach.

**Key Technologies:**

- C++20 standard
- Cross-platform support (Windows, Linux, macOS, Android, iOS)
- MiniAudio as the default audio device implementation
- XMake build system
- Plugin architecture for extensibility

## Code Style and Formatting

### General Guidelines

- **Standard:** C++20
- **Column Limit:** 140 characters
- **Indentation:** 4 spaces (never tabs)
- **Line Endings:** Keep empty lines to a maximum of 1

### Naming Conventions

- **Classes/Structs:** PascalCase (e.g., `Engine`, `AudioBuffer`, `HRIRSphere`)
- **Functions/Methods:** PascalCase (e.g., `Initialize()`, `GetInstance()`, `LoadSoundBank()`)
- **Variables:** camelCase (e.g., `sampleRate`, `channelCount`)
- **Constants:** PascalCase or SCREAMING_SNAKE_CASE for preprocessor defines
- **Namespaces:** PascalCase, fully indented
- **Macros:** Prefixed with `AM_` or `am` (e.g., `AM_API_PUBLIC`, `amEngine`)

### Formatting Details

- **Braces:** Allman style (braces on new lines for classes, functions, control statements)
- **Pointer/Reference Alignment:** Left-aligned (e.g., `Type* ptr`, `Type& ref`)
- **Template Declarations:** Always break after template keyword
- **Function Parameters:** Break after open bracket if needed, one per line
- **Inheritance Lists:** Break before colon and commas
- **Constructor Initializers:** Break before comma

### Header Files

- Always use include guards with pattern: `#ifndef _AM_MODULE_FILE_H` / `#define _AM_MODULE_FILE_H` / `#endif`
- Include standard copyright header:

```cpp
// Copyright (c) [CURRENT_YEAR]-present Sparky Studios. All rights reserved.
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.
```

- Header path structure: `SparkyStudios/Audio/Amplitude/[Module]/[File].h`

### Documentation

- Use Doxygen comments for all public APIs
- Include `@brief` descriptions for classes and functions
- Use `@ingroup` tags to organize documentation
- Document parameters with `@param`, return values with `@return`
- Reference code examples in documentation with `@code{cpp}` blocks
- Link to documentation site when relevant (https://docs.amplitudeaudiosdk.com/nightly/)

## Architecture Guidelines

### Module Organization

The SDK is organized into major modules:

- **Core:** Engine, entities, listeners, rooms, memory management, threading
- **Sound:** Sound objects, collections, switches, effects, RTPC, attenuation
- **Mixer:** Audio pipeline, nodes, Amplimix processor
- **IO:** File systems, logging, resources (disk, memory, package, platform-specific)
- **DSP:** Filters, FFT, resampling, audio conversion, convolution
- **Math:** Linear algebra, geometry, curves, spatial positioning
- **HRTF:** Binaural audio processing

### Design Patterns

- **Singleton:** Engine class uses singleton pattern (accessed via `amEngine` macro)
- **Plugin Architecture:** Drivers, codecs, and other components support plugin extensions
- **Handle Types:** Use typedef'd pointers for major types (e.g., `SoundHandle`, `EventHandle`)
- **Reference Counting:** Used for resource management
- **Platform Abstraction:** Platform-specific code isolated in `Platforms/` subdirectories

### Memory Management

- Custom memory management system available
- Use appropriate allocation strategies for audio buffers
- Consider memory pools for frequently allocated objects

### Threading

- Engine uses STL mutex for synchronization
- Thread-safe operations where appropriate
- Background audio processing considerations

## Testing

### Test Framework

- Tests located in `tests/` directory, organized by module (e.g., `tests/core_engine/`, `tests/dsp_*/`, `tests/math_*/`)
- Each test is a separate `.cpp` file that self-registers via static initialization
- XMake configuration at `tests/xmake.lua` with platform-specific runners in `tests/runners/`
- Tests cover core functionality: engine, audio buffers, DSP, math, threading, HRTF, etc.

### Running Tests

- **Run all tests:** `xmake test` (this automatically builds the SDK and test targets first -- no separate build step needed)
- **Configure for tests:** `xmake f --unit_tests=y` (must be enabled before running tests)
- **Build tests only:** `xmake build amplitude_tests`
- **List all tests:** `./amplitude_tests -l`
- **Run with verbose output:** `./amplitude_tests -v`
- **Run a specific test group:** `./amplitude_tests -g core_engine`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AmplitudeAudio/sdk](https://github.com/AmplitudeAudio/sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
