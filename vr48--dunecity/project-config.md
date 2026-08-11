---
trigger: always_on
description: - Follow `.analysis/development-approach.md` (2-doc feature process under `.analysis/features/`).
---

# Dune Legacy Project Configuration for Cursor

## Workflow
- Follow `.analysis/development-approach.md` (2-doc feature process under `.analysis/features/`).

## Project Overview
This is Dune Legacy, a modernized clone of Westwood Studios' "Dune 2" real-time strategy game.

### Current Development
- **Version**: 0.99.5 (see `CMakeLists.txt` line 4)
- **Path**: `/Users/stefanvanderwel/development/dune/dunelegacy/`
- Platform: Cross-platform (Windows, macOS, Linux)
- Language: C++17
- Graphics: SDL2, SDL2_mixer, SDL2_ttf

## Development Environment

### Windows Development
- **Shell**: PowerShell (default on Windows)
- **Terminal**: Windows Terminal or PowerShell
- **Commands**: Use PowerShell syntax for Windows-specific operations
- **Path Separator**: Use backslashes (`\`) for Windows paths in PowerShell commands
- **Script Execution**: PowerShell scripts use `.ps1` extension

### macOS Development
- **Shell**: Bash/Zsh
- **Terminal**: Terminal.app or iTerm2

### Linux Development
- **Shell**: Bash
- **Terminal**: Various terminal emulators

## Build System & Platforms

### macOS (Primary Development Platform)
- **IDE**: Xcode project located at `IDE/xCode/Dune Legacy.xcodeproj`
- **Build Command**: `cd IDE/xCode && xcodebuild -project "Dune Legacy.xcodeproj" -configuration Release`
- **Output**: `IDE/xCode/build/Release/Dune Legacy.app`
- **Installer Creation**: `./create_dmg.sh` (creates DMG from Xcode build)
- **Dependencies**: SDL2 frameworks included in `IDE/xCode/external/frameworks/`

### Windows
- **Shell**: PowerShell (use PowerShell syntax for commands)
- **Cross-compilation**: `./buildcrosswin32.sh` (requires mingw, run in PowerShell)
- **Native**: Visual Studio project in `IDE/VC/`
- **Installer**: NSIS scripts in `legacy/nsis-old/` directory
- **Build Commands**: Use PowerShell syntax, e.g., `.\buildcrosswin32.sh` or `& .\buildcrosswin32.sh`
- **Visual Studio 2022**: Open `IDE/VC/DuneLegacy.sln` and build for x64 Release/Debug
- **SDL Dependencies**: All SDL2 libraries are included as project references in the solution

### Linux
- **Build System**: CMake or Autotools
- **Commands**: 
  - CMake: `cmake -B build && make -C build`
  - Autotools: `autoreconf --install && ./configure && make`

## Key Dependencies
- SDL2 (≥2.32.4)
- SDL2_mixer (≥2.8.1) 
- SDL2_ttf (≥2.24.0)
- Original Dune 2 PAK files (required for gameplay)

## Important Directories
- `src/` - Main source code
- `include/` - Header files
- `data/` - Game data files and PAK files
- `IDE/` - Platform-specific IDE projects
- `tests/` - Unit tests
- `nsis/` - Windows installer scripts

## Common Tasks

### Building for macOS

#### Quick Build + Installer (RECOMMENDED)
**Always use the CMake build system with vcpkg for consistent, reproducible builds:**

```bash
# Navigate to project root
cd /Users/stefanvanderwel/development/dune/dunelegacy

# Build and create DMG installer (one command does it all!)
cmake --build build --target dmg --config Release
```

**Output**: `build/DuneLegacy-0.99.5-macOS.dmg`

**What this does automatically:**
1. Cleans CPack cache to prevent stale files
2. Builds the binary with Release optimizations
3. Stages all files (app bundle, data, config, docs)
4. Creates fresh DMG installer with hdiutil
5. Cleans up staging directory

#### Fresh Build from Scratch (Clean Build)
**Use this when you want to rebuild everything without using any cache:**

```bash
cd /Users/stefanvanderwel/development/dune/dunelegacy

# Step 1: Remove build directory
rm -rf build

# Step 2: Configure CMake with vcpkg (builds all dependencies from source)
cmake -B build \
  -DCMAKE_TOOLCHAIN_FILE=/Users/stefanvanderwel/development/dune/vcpkg/scripts/buildsystems/vcpkg.cmake \
  -DCMAKE_BUILD_TYPE=Release

# Step 3: Build and create DMG installer
cmake --build build --target dmg --config Release
```

**First build takes ~5-10 minutes** as vcpkg downloads and compiles SDL2 libraries.
**Subsequent builds are much faster** (seconds to minutes depending on changes).

#### Legacy Xcode Build (Alternative)
```bash
cd IDE/xCode
xcodebuild -project "Dune Legacy.xcodeproj" -configuration Release
./create_dmg.sh  # Creates DMG from Xcode build
```

### Building for Windows

#### CMake + vcpkg Build (RECOMMENDED)
```powershell
# PowerShell - Navigate to project root
cd C:\path\to\dunelegacy

# Configure (one-time or after CMakeLists.txt changes)
cmake -B build -G "Visual Studio 17 2022" -A x64 `
  -DCMAKE_TOOLCHAIN_FILE=C:\vcpkg\scripts\buildsystems\vcpkg.cmake

# Build and create installer
cmake --build build --target installer --config Release
```

**Output**: 
- `build/DuneLegacy-0.99.5-Windows-x64.exe` (NSIS installer)
- `build/DuneLegacy-0.99.5-Windows-x64.zip` (portable zip)

### Building for Linux

```bash
cd /path/to/dunelegacy

# Configure with vcpkg
cmake -B build \
  -DCMAKE_TOOLCHAIN_FILE=/path/to/vcpkg/scripts/buildsystems/vcpkg.cmake \
  -DCMAKE_BUILD_TYPE=Release

# Build and create packages (DEB, RPM, TGZ)
cmake --build build --target package --config Release
```

### Running Tests

Unit tests use the Catch2 framework. See `tests/README.md` for full documentation.

```bash
# Install Catch2 (one-time setup)
brew install catch2  # macOS
# sudo apt-get install catch2  # Debian/Ubuntu


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VR48/dunecity](https://github.com/VR48/dunecity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
