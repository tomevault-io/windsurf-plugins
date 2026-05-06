---
trigger: always_on
description: **Documentation:** See `documentation/AGENTS.md` for agent discoverability and `documentation/TESTING.md` for testing.
---

# Qt MCP Plugin - Cursor AI Build Instructions

## Project Overview

**Documentation:** See `documentation/AGENTS.md` for agent discoverability and `documentation/TESTING.md` for testing.
This is a Qt Creator plugin that implements the Model Context Protocol (MCP) for AI control of Qt Creator.

---
## ⚠️ STEP 1: Configure Your Qt Installation Path ⚠️

**Qt paths are stored in `.qt/qt_path_{platform}.txt` files in the project root.**

### Files:
- `.qt/qt_path_windows.txt` - Windows Qt path
- `.qt/qt_path_darwin.txt` - macOS Qt path  
- `.qt/qt_path_linux.txt` - Linux Qt path

### Auto-Discovery:
The build script will try to auto-discover Qt from common locations:
- Windows: `%USERPROFILE%\Developer\Qt`, `C:\Qt`
- macOS: `~/Developer/Qt`, `/Applications/Qt`
- Linux: `~/Qt`, `/opt/Qt`

If found, the path is saved to the config file automatically.

### Manual Configuration:
If auto-discovery fails, edit the appropriate file and paste your Qt path:
```
# Example for Windows (.qt/qt_path_windows.txt):
C:\Users\username\Developer\Qt

# Example for macOS (.qt/qt_path_darwin.txt):
/Users/username/Developer/Qt
```

### Finding Your Qt Path:
```powershell
# Windows - look for folder containing Tools\QtCreator\:
dir "$env:USERPROFILE\Developer\Qt"
dir "C:\Qt"

# macOS - look for folder containing Qt Creator.app:
ls ~/Developer/Qt/
ls /Applications/
```

**QT_ROOT** = The folder containing `Tools\QtCreator\` (Windows) or `Qt Creator.app` (macOS)

---
## ⚠️ STEP 2: Detect Qt Creator's Qt Version ⚠️

**The Qt SDK version MUST EXACTLY MATCH what Qt Creator was built with.**

### How to detect Qt Creator's Qt version (THE CORRECT WAY):

**Option 1 - Use qtdiag (RECOMMENDED - cross-platform):**
```bash
# Windows:
$QT_ROOT\Tools\QtCreator\bin\qtdiag.exe

# macOS:
$QT_ROOT/Qt\ Creator.app/Contents/MacOS/qtdiag

# Look for line like: "Qt 6.10.1 (x86_64-little_endian-lp64 shared (dynamic) release build; by MSVC 2022)"
```

**Option 2 - Qt Creator About dialog:**
- Open Qt Creator → Help → About Qt Creator
- Look for: "Based on Qt 6.10.1 (MSVC 2022, x86_64)"

**Option 3 - Command line:**
```bash
qtcreator --version
# Output: "Qt Creator 15.0.0 based on Qt 6.10.1 (MSVC 2022, x86_64)"
```

### ⛔ DO NOT read version from QtCreatorConfig.cmake!
The CMake file contains `find_dependency(Qt6 "X.Y.Z")` but this is a **MINIMUM version**, 
not the actual version. Using this will give you the WRONG version!

### What Qt SDK to install:
1. Get the Qt version from qtdiag (e.g., "6.10.1")
2. Get the compiler from qtdiag (e.g., "MSVC 2022" or "Apple Clang")
3. Install that EXACT version with matching compiler:
   - Windows: Qt X.Y.Z → msvc2022_64 (or msvc2019_64 if that's what qtdiag shows)
   - macOS: Qt X.Y.Z → macos

---
## 🟢 HOW TO BUILD 🟢

**To build and install the plugin, run:**
```bash
python3 scripts/build/build_main.py
```

This script handles EVERYTHING automatically:
- ✅ Quits Qt Creator if running
- ✅ Detects Qt version from Qt Creator
- ✅ Verifies Qt SDK is installed (prompts if missing)
- ✅ Cleans old plugin versions
- ✅ Builds with correct Qt paths
- ✅ Installs to Qt Creator
- ✅ Launches Qt Creator
- ✅ Tests MCP server
- ✅ Registers with Cursor IDE

**DO NOT manually run cmake or build commands. Always use the build script.**

### ⚠️ After every atomic change: run the build script
**Anytime you make an atomic change to the plugin** (source code, CMake, version, JSON, discovery file, resources, etc.), **you MUST run the build script** (`python3 scripts/build/build_main.py`). The script will quit Qt Creator, bump the version, build, install, launch Qt Creator, and verify the new plugin version. Do not skip this step after editing plugin-related files.

---
## 🔴 MANUAL: Verify SDK Installation (if build script fails) 🔴

**The build script does this automatically. Only use manually if troubleshooting.**

### Step 1: Get Qt version from qtdiag (see CRITICAL section above)
Use `qtdiag` or `qtcreator --version` to find the EXACT Qt version.

### Step 2: Check installed Qt SDKs
```bash
# Windows:
dir C:\Users\%USERNAME%\Developer\Qt

# macOS:
ls -la ~/Developer/Qt/
```
Look for folders like `6.10.1`, `6.11.0`, etc.

### Step 3: Verify compiler variant exists
```bash
# Windows - need msvc2022_64 or msvc2019_64:
dir C:\Users\%USERNAME%\Developer\Qt\6.10.1\

# macOS - need macos:
ls ~/Developer/Qt/6.10.1/
```

### Step 4: If Qt SDK is Missing - STOP AND NOTIFY USER
**If the required Qt version is NOT installed:**
```
STOP: Qt SDK version X.Y.Z is required but not installed.

Qt Creator uses Qt X.Y.Z but you have: [list installed versions]

ACTION REQUIRED:
1. Run MaintenanceTool.exe (Windows) or MaintenanceTool.app (macOS)
2. Select "Add or remove components"  
3. Install Qt X.Y.Z with:
   - Windows: MSVC 2022 64-bit (msvc2022_64) or MSVC 2019 64-bit (msvc2019_64)
   - macOS: macOS
4. Re-run the build after installation completes
```

**DO NOT PROCEED WITH BUILD until the correct Qt SDK is installed.**

---
## ⚠️ CRITICAL: Plugin Installation Requirements ⚠️

**YOU MUST FOLLOW THESE STEPS IN ORDER - NO EXCEPTIONS:**

1. **QUIT Qt Creator** - The plugin cannot be updated while Qt Creator is running

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [davecotter/Qt-Creator-MCP-Plugin](https://github.com/davecotter/Qt-Creator-MCP-Plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
