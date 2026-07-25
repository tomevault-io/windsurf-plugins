---
trigger: always_on
description: Supercell Wx is a cross-platform C++20/Qt6 application for visualizing live and archived NEXRAD weather radar data. This guide helps AI agents understand the codebase architecture and development workflow.
---

# Supercell Wx AI Agent Instructions

Supercell Wx is a cross-platform C++20/Qt6 application for visualizing live and archived NEXRAD weather radar data. This guide helps AI agents understand the codebase architecture and development workflow.

## Project Architecture

### Two-Library Structure
- **wxdata/** - Core radar data processing library (platform-independent, no Qt)
  - Parses NEXRAD Level 2/3 files ([wxdata/include/scwx/wsr88d/](wxdata/include/scwx/wsr88d/))
  - Network providers for AWS/NWS data ([wxdata/include/scwx/provider/](wxdata/include/scwx/provider/))
  - AWIPS message parsing ([wxdata/include/scwx/awips/](wxdata/include/scwx/awips/))
  - GR placefile support ([wxdata/include/scwx/gr/](wxdata/include/scwx/gr/))
  - Uses shared Conan dependencies but NO Qt

- **scwx-qt/** - Qt GUI application layer
  - Main window and UI ([scwx-qt/source/scwx/qt/main/](scwx-qt/source/scwx/qt/main/))
  - Manager classes coordinate application state ([scwx-qt/source/scwx/qt/manager/](scwx-qt/source/scwx/qt/manager/))
  - Map rendering with MapLibre GL ([scwx-qt/source/scwx/qt/map/](scwx-qt/source/scwx/qt/map/))
  - OpenGL drawing primitives ([scwx-qt/source/scwx/qt/gl/](scwx-qt/source/scwx/qt/gl/))
  - Product views connect data to visualization ([scwx-qt/source/scwx/qt/view/](scwx-qt/source/scwx/qt/view/))

**Critical:** Keep Qt code isolated to scwx-qt. Never add Qt dependencies to wxdata.

### Manager Pattern
Manager classes in [scwx-qt/source/scwx/qt/manager/](scwx-qt/source/scwx/qt/manager/) are singletons that manage global application concerns:
- `RadarProductManager` - loads/caches radar products, emits Qt signals for data updates
- `SettingsManager` - persistent settings via QSettings
- `AlertManager` - weather alert processing
- `PlacefileManager` - external placefile integration
- `TimelineManager` - time-based product selection

Managers communicate via Qt signals/slots. When data flows from wxdata → scwx-qt, it typically goes through a manager.

### Background Task Processing
Background tasks use **Boost thread pools and ASIO post** for asynchronous operations. This includes network requests, file I/O, and data processing that shouldn't block the UI thread.

### Namespace Convention
All code uses nested `namespace scwx { namespace X { ... } }` structure:
- `scwx::wsr88d` - NEXRAD data structures
- `scwx::qt::view` - Qt view classes
- `scwx::qt::manager` - manager singletons
Use fully qualified namespaces in headers; `using namespace` prohibited in headers per Google C++ Style Guide.

## Build System

### Conan + CMake Workflow
The project uses **Conan 2** for C++ dependency management. CMake integrates Conan via `cmake-conan` provider.

**Setup script usage** (recommended path for new developers):
```powershell
# Windows (from repo root)
.\tools\setup-windows-vs2026-x64-release.bat [BUILD_DIR] [VENV_PATH]

# Linux
./tools/setup-linux-ninja-release.sh [BUILD_DIR] [CONAN_PROFILE] [VENV_PATH] [ASAN_ENABLE]
```

**Manual CMake configuration:**
```bash
# 1. Install Conan profile
conan config install ./tools/conan/profiles/scwx-windows_vs2026_x64 -tf profiles

# 2. Install dependencies
mkdir build && cd build
conan install ../
  --remote conancenter
  --build missing
  --profile:all scwx-windows_vs2026_x64
  --settings:all build_type=Release
  --output-folder ./conan/

# 3. Configure CMake (Conan provider auto-installs deps if conan/ exists)
cmake ../ -G Ninja
  -DCMAKE_BUILD_TYPE=Release
  -DCMAKE_PROJECT_TOP_LEVEL_INCLUDES=../external/cmake-conan/conan_provider.cmake
  -DCONAN_HOST_PROFILE=scwx-windows_vs2026_x64
  -DCONAN_BUILD_PROFILE=scwx-windows_vs2026_x64

# 4. Build
cmake --build . --target supercell-wx
```

**Key Conan profiles:** See [tools/conan/profiles/](tools/conan/profiles/)
- Windows: `scwx-windows_vs2026_x64[-debug]`
- Linux: `scwx-linux_gcc-11[-debug]`, `scwx-linux_clang-17`
- macOS: `scwx-macos_clang-18[_armv8][-debug]`

**CMake Presets:** Use [CMakePresets.json](CMakePresets.json) for IDE integration. Presets like `windows-vs2026-x64-release` encapsulate toolchain/profile selection.

### Python Virtual Environment
Project uses Python for code generation (counties DB, version info). Setup scripts create `.venv/` with requirements from [requirements.txt](requirements.txt). CMake macro `scwx_python_setup()` in [tools/scwx_config.cmake](tools/scwx_config.cmake) finds the venv Python.

### Build Outputs
Per [tools/scwx_config.cmake](tools/scwx_config.cmake), binaries go to:
- `build/<preset>/<BuildType>/bin/supercell-wx[.exe]`
- `build/<preset>/<BuildType>/lib/` for shared libraries

## External Dependencies

### Vendored vs Conan
- **Conan-managed** ([conanfile.py](conanfile.py)): Boost, Qt (via system), GEOS, libcurl, OpenSSL, spdlog, SQLite, etc.
- **Git submodules** ([external/](external/)): MapLibre Native Qt, ImGui, stb, units library
  - Use submodules when heavy customization or unreleased versions needed
  - MapLibre is vendored because Qt bindings require custom build

### Qt 6.11.1 Requirement

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dpaulat/supercell-wx](https://github.com/dpaulat/supercell-wx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
