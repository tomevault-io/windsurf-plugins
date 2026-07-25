---
trigger: always_on
description: 8-Bit Analysers is a suite of GUI tools for analyzing and annotating games on 8-bit systems (ZX Spectrum, C64, CPC, BBC Micro, Arcade Z80, Tube Elite). Each analyser is a separate CMake project sharing a large common library (C++17).
---

# 8-Bit Analysers - Copilot Instructions

## Project Overview

8-Bit Analysers is a suite of GUI tools for analyzing and annotating games on 8-bit systems (ZX Spectrum, C64, CPC, BBC Micro, Arcade Z80, Tube Elite). Each analyser is a separate CMake project sharing a large common library (C++17).

**Key architectural principle**: The shared library is **not** compiled separately—it's compiled directly into each application binary via `CMakeShared.txt`.

## Build System

Each analyser lives under `Source/<AnalyserName>/` (ZXSpectrum, ArcadeZ80, C64, CPC, BBC, TubeElite). Build steps are identical:

```bash
cd Source/ArcadeZ80   # or ZXSpectrum, C64, CPC, BBC, TubeElite
mkdir build && cd build
cmake ..
```

**Platform-specific:**
- **Linux/macOS**: `make`
- **macOS (Xcode)**: `cmake -G Xcode ..`
- **Windows**: Open the generated `.sln` in `build/`

**Output:** Binaries go to `build/bin/`. On Windows, the VS debugger working directory is set to `../../Data/<AnalyserName>` in the CMake project.

**First run setup:**
1. Copy `Data/<AnalyserName>/imgui.ini` to your working directory
2. Edit the generated `globalconfig.json` to set `WorkspaceRoot`, `SnapshotFolder`, and `PokesFolder` paths

**Clone requirements:**
```bash
git clone --recursive https://github.com/TheGoodDoktor/8BitAnalysers
```
All vendor dependencies are git submodules.

### CMakeShared.txt

`Source/Shared/CMakeShared.txt` defines the `shared_src` variable by globbing all shared source files. It's included by each platform's CMakeLists.txt and selects platform-specific implementations:

- **GLFW backend** (all platforms): `ImGuiSupport/GLFW/*.cpp/.h`
- **DX11 backend** (Windows only, currently unused): `ImGuiSupport/Windows/*.cpp/.h`
- **Platform utils**:
  - Windows: `Util/Windows/FileUtil_Win32.cpp`
  - Linux: `Util/Linux/FileUtil_Linux.cpp`
  - macOS: `Util/Mac/FileUtil_Mac.mm` (Objective-C++)

## Testing

Tests are **disabled by default**. To enable:

1. In `Source/<AnalyserName>/CMakeLists.txt`, set:
   ```cmake
   set( with_tests true )
   ```

2. Rebuild and run:
   ```bash
   ctest --test-dir build
   # or run directly:
   ./build/bin/SpectrumAnalyserTest
   ```

**Test framework:** Google Test v1.13.0 (fetched via FetchContent when `with_tests=true`)

**Test locations:**
- Shared tests: `Source/Shared/CodeAnalyser/Tests/CodeAnalyserTests.cpp`
- Platform tests: `Source/ZXSpectrum/Tests/SpectrumTests.cpp`

**Test patterns:**
- Basic assertions: `TEST(ZXSpectrumTest, BasicAssertions)`
- Emulator fixtures: `TEST_F(FSpectrumEmuTest, SnapshotLoaderTest)`
- Tests use `TEST` compile definition

**macOS note:** Tests are force-disabled on macOS in CMakeLists.txt (line 45)

## Architecture

### Multi-Analyser Structure

Each analyser app is independently buildable and links against the shared library. The shared library (`Source/Shared/`) is **not** a separate `.so`/`.dll` — it's compiled directly into each app via `CMakeShared.txt` which globs all shared source files.

### Key Subsystems (`Source/Shared/`)

#### **CodeAnalyser/** — Core analysis engine
- `FCodeAnalyser` owns analysis state, manages memory pages (`FCodeAnalysisPage`), and drives the disassembly pipeline
- CPU-specific disassemblers: `CodeAnalyser/Z80/` and `CodeAnalyser/6502/`
- **Commands/** — Undo/redo system using command pattern
  - `CommandProcessor` manages undo stack
  - Command types: `SetItemDataCommand`, `SetCommentCommand`, `FormatDataCommand`

#### **Misc/EmuBase.h** — Emulator base class
- `FEmuBase` is the base class all platform emulators derive from (e.g., `FArcadeZ80` in `Source/ArcadeZ80/`)
- Owns the `FCodeAnalyser` instance
- Wires up main loop via `Init()`, `Tick()`, `Shutdown()`
- Handles UI drawing through `DrawUI()`, `DrawMainMenu()`, `DrawDockingView()`

#### **MCPServer/** — Model Context Protocol server
- Exposes analyser state to LLMs
- `MCPServer.cpp` — Server lifecycle management
- `MCPTools.cpp` — Callable tools definition
- `MCPTransport.cpp` — Communication layer
- Controlled via `bRunMCPServer` flag in launch config

#### **LuaScripting/** — Lua integration
- `LuaConsole` — Interactive console UI
- `LuaCoreAPI` and `LuaSys` — Expose analyser internals to Lua scripts
- Used for user automation and scripting

#### **CodeAnalyser/UI/** — ImGui rendering
- All analyser UI components
- CPU-specific UI: `UI/Z80/` and `UI/6502/`

#### **ImGuiSupport/** — Platform backends
- `GLFW/` — Active backend on all platforms (Windows, Linux, macOS)
- `Windows/` — DX11 option (currently unused)
- Graphics API controlled via `gfxapi` CMake variable

#### **Util/** — Platform-specific utilities
- `Windows/`, `Linux/`, `Mac/` subdirs selected at compile time
- File I/O, memory utilities, graphics helpers
- **macOS**: Uses `.mm` Objective-C++ files

### Vendor Dependencies (`Source/Vendor/`)

All managed as git submodules:
- `chips` — CPU emulation cores (Z80/6502)
- `imgui` — GUI framework
- `glfw` — Windowing/input
- `lua` — Scripting engine
- `nlohmann/json` — JSON parsing
- `implot` — Plotting library
- `magic_enum` — Enum reflection
- `ImGuiColorTextEdit` — Code editor widget


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheGoodDoktor/8BitAnalysers](https://github.com/TheGoodDoktor/8BitAnalysers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
