---
trigger: always_on
description: Gearmulator emulates classic virtual analog synthesizers from the late 90s/2000s by emulating the original ICs and running authentic firmware (ROMs) as audio plugins. Supported devices include Access Virus (A/B/C/TI/Snow), Waldorf microQ/Microwave II/XT, Clavia Nord Lead/Rack 2x, and Roland JP-8000.
---

# Gearmulator Development Guide

Gearmulator emulates classic virtual analog synthesizers from the late 90s/2000s by emulating the original ICs and running authentic firmware (ROMs) as audio plugins. Supported devices include Access Virus (A/B/C/TI/Snow), Waldorf microQ/Microwave II/XT, Clavia Nord Lead/Rack 2x, and Roland JP-8000.

## Build System

### CMake Options

Configure which synths to build:
- `gearmulator_SYNTH_OSIRUS` - Access Virus A/B/C (default: on)
- `gearmulator_SYNTH_OSTIRUS` - Access Virus TI/TI2/Snow (default: on)
- `gearmulator_SYNTH_VAVRA` - Waldorf microQ (default: on)
- `gearmulator_SYNTH_XENIA` - Waldorf Microwave II/XT (default: on)
- `gearmulator_SYNTH_NODALRED2X` - Clavia Nord Lead/Rack 2x (default: on)
- `gearmulator_SYNTH_JE8086` - Roland JP-8000 (default: on)

Configure plugin formats:
- `gearmulator_BUILD_JUCEPLUGIN` - Build JUCE-based plugins (default: on)
- `gearmulator_BUILD_JUCEPLUGIN_CLAP` - Build CLAP format (default: on)
- `gearmulator_BUILD_JUCEPLUGIN_LV2` - Build LV2 format (default: on)
- `gearmulator_BUILD_FX_PLUGIN` - Build FX variants (default: off)

### Build Commands

**Windows:**
The combined Maschine MD-MM target is `mdmmJucePlugin_Standalone`. A Windows
release packager for this product has not been established yet.

**Current Development Setup:**
- The macOS release build uses `build/macos-mdmm-universal` by default.
- The combined app target is `mdmmJucePlugin_Standalone`.

**Linux:**
```bash
cmake . -B temp/cmake_linux -Dgearmulator_BUILD_JUCEPLUGIN=ON -Dgearmulator_BUILD_JUCEPLUGIN_CLAP=ON
cmake --build temp/cmake_linux --config Release -j 4
cd temp/cmake_linux && cpack -G DEB  # or RPM or ZIP
```

**macOS:**
```bash
scripts/macos/build_mdmm.sh
scripts/macos/build_installer.sh
```

The macOS scripts build and package Maschine MD-MM with the separate MD/MM
products. `build_linux.sh` remains an upstream general-purpose helper.

### Tests

Run integration tests:
```bash
ctest -C Release
```

Individual test consoles exist for each synth:
- `virusIntegrationTest` - Virus integration tests
- `virusTestConsole` - Virus test console
- `mqTestConsole` - Waldorf microQ test console
- `xtTestConsole` - Waldorf Microwave II/XT test console
- `n2xTestConsole` - Nord Lead 2x test console
- `jeTestConsole` - JP-8000 test console

## Architecture

### Core Emulation Layers

1. **DSP Emulation** (`dsp56300/`)
   - Motorola DSP 56300 family emulator at the core
   - Used by Access Virus, Waldorf Q/microQ, Nord Lead 3, Novation Supernova
   - JIT compilation via asmjit for performance
   - Peripherals: ESAI, HDI08 (Host Device Interface)

2. **MC68K Emulation** (`mc68k/`)
   - Motorola 68000 family emulator using Musashi
   - Used by devices requiring 68K microcontrollers

3. **Device Libraries** (per-synth)
   - `virusLib/` - Access Virus A/B/C/TI/Snow implementation
   - `mqLib/` - Waldorf microQ implementation
   - `xtLib/` - Waldorf Microwave II/XT implementation
   - `nord/n2x/` - Clavia Nord Lead/Rack 2x implementation
   - `ronaldo/je8086/` - Roland JP-8000 implementation

4. **Hardware Abstraction** (`hardwareLib/`)
   - Common hardware peripherals (LCD, buttons, encoders)
   - Hardware interface abstractions

5. **Common Libraries**
   - `baseLib/` - Low-level utilities (filesystem, logging, events, binary streams)
   - `synthLib/` - Cross-synth audio plugin infrastructure (Device base class, DAC, resampling, MIDI routing, parameter system)

### Plugin Architecture

Each synth has a JUCE plugin implementation:
- `osirusJucePlugin/` - Osirus (Virus A/B/C) plugin
- `osTIrusJucePlugin/` - OsTIrus (Virus TI) plugin
- `mqJucePlugin/` - Vavra (microQ) plugin
- `xtJucePlugin/` - Xenia (Microwave II/XT) plugin
- `nord/n2x/n2xJucePlugin/` - Nodal Red 2x (Nord Lead) plugin
- `ronaldo/je8086/jeJucePlugin/` - JE-8086 (JP-8000) plugin

**Key Plugin Components:**
- `jucePluginLib/` - Core JUCE plugin infrastructure (Parameter system, processor base, MIDI handling, controller mapping)
- `juceRmlUi/` - RmlUi integration for declarative UI (HTML/CSS-like)
- `juceUiLib/` - Common UI components
- `jucePluginEditorLib/` - Plugin editor infrastructure
- `jucePluginData/` - Shared plugin assets

**Plugin Build Flow:**
1. Each synth plugin calls `createJucePluginWithFX()` macro from `juce.cmake`
2. Links device-specific library (e.g., `virusJucePlugin` links `virusLib`)
3. Processor inherits from `synthLib::Plugin` which wraps `synthLib::Device`
4. Skins defined via RML/RCSS files, compiled into binary data
5. Multiple skins per synth using `addSkin()` and `buildSkinHeader()` macros

### Device Model Pattern

All synth implementations follow this pattern:

```cpp
namespace <synthName>Lib {
    class Device : public synthLib::Device {
        // DSP/MC68K instance
        // Microcontroller for MIDI/UI
        // ROM loading and memory setup
        // Audio I/O via HDI08 or similar
    };
}
```

Key responsibilities:
- Load firmware ROM files
- Initialize DSP/CPU memory
- Handle MIDI via microcontroller or direct DSP communication

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mo0kid/maschine-md-mm](https://github.com/mo0kid/maschine-md-mm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
