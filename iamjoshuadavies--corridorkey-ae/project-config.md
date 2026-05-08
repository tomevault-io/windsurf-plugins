---
trigger: always_on
description: Native Adobe After Effects plugin for green-screen keying, based on CorridorKey.
---

# CorridorKey AE - Claude Code Context

## Project Overview
Native Adobe After Effects plugin for green-screen keying, based on CorridorKey.
Three-layer architecture: Host Plugin (C++) → Bridge (IPC) → Runtime (Python/MLX).
Created by Niko Pueringer / Corridor Digital. AE plugin wrapper by this project.

## Build Commands

### macOS
```bash
# Configure and build plugin — must specify SDK path
cmake -B build -S . -DCMAKE_BUILD_TYPE=Debug -DAE_SDK_PATH=/Users/schwar/Documents/corridorkey-ae/ae_sdk
cmake --build build

# Clean rebuild (needed when PiPL or flags change — AE caches aggressively)
rm -rf build && cmake -B build -S . -DCMAKE_BUILD_TYPE=Debug -DAE_SDK_PATH=/Users/schwar/Documents/corridorkey-ae/ae_sdk && cmake --build build

# Run runtime tests
cd runtime && source .venv/bin/activate && python -m pytest tests/

# Run full test suite
scripts/bootstrap/run_tests.sh

# Start runtime manually (for dev — auto-launch handles this normally)
cd runtime && source .venv/bin/activate && python -m server.main --port 12345
```

### Windows
```bash
# Configure with VS 2019 generator (matches BuildTools 2019 on this machine).
# AE_SDK_PATH points at the Examples folder of the unpacked Win SDK; CMake
# also auto-detects a nested ae_sdk_win/<zip>/<zstd>/Examples layout if you
# don't pass it explicitly.
cmake -B build_win -S . -G "Visual Studio 16 2019" -A x64 \
      -DAE_SDK_PATH="C:/Users/iamjo/Documents/corridorkey-ae/ae_sdk_win/AfterEffectsSDK_25.6_61_win/ae25.6_61.64bit.AfterEffectsSDK/Examples"
cmake --build build_win --config Release

# Install the .aex into AE's plug-ins folder. Program Files needs admin —
# run this from an elevated PowerShell with AE closed (the file is locked
# while AE is open).
Copy-Item -Force "build_win/plugin/Release/CorridorKey.aex" \
  "C:/Program Files/Adobe/Adobe After Effects 2026/Support Files/Plug-ins/Effects/CorridorKey.aex"

# Bridge discovers the runtime venv via:
#   1. CORRIDORKEY_REPO_ROOT env var      (dev escape hatch)
#   2. %LOCALAPPDATA%\CorridorKey         (per-user install)
#   3. %ProgramFiles%\CorridorKey         (system-wide install)
# For a source-tree dev workflow set the env var once:
[Environment]::SetEnvironmentVariable('CORRIDORKEY_REPO_ROOT', 'C:\Users\iamjo\Documents\corridorkey-ae', 'User')

# Runtime venv (Windows PyTorch + CUDA + model deps)
cd runtime
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install msgpack numpy Pillow opencv-python-headless timm safetensors
.\.venv\Scripts\python.exe -m pip install torch==2.5.1+cu121 torchvision==0.20.1+cu121 --index-url https://download.pytorch.org/whl/cu121

# Start runtime manually
.\.venv\Scripts\python.exe -m server.main --port 12345
```

Claude's bash session is not elevated even when Claude Desktop is launched
as admin (sandbox runs at medium integrity). The `Copy-Item` step into
Program Files must be run by the user from an admin shell.

## Installers (feature/installers branch — WIP, issue #28)

**Status:** Both platforms done end-to-end. macOS `.pkg` via
`pkgbuild`/`productbuild`, Windows `.exe` via InnoSetup 6. Both build
from the same plugin artifact in CI, both ship
python-build-standalone + runtime source + requirements and create
the venv on the user's machine at install time. Work lives on the
`feature/installers` branch, not yet merged to main — waiting until
both platforms are verified in the wild.

### Done (macOS)
- `scripts/installer/build_macos.sh` — stages `python-build-standalone`
  3.12.13 aarch64 + runtime source + plugin bundle + `requirements.txt`
  into a temp tree, runs `pkgbuild` + `productbuild`, drops a
  ~17 MB `CorridorKey-<ver>-macOS-arm64.pkg` in `dist/`.
- `installer/macos/Distribution.xml` — productbuild wrapper.
  `localSystem` domain (NOT per-user) because AE on macOS only scans
  5 system paths under `/Applications/Adobe After Effects */Plug-Ins/`,
  so there's no friction-free user install. One admin prompt at
  install time is unavoidable.
- `installer/macos/scripts/postinstall` — runs as root after payload
  is laid down at `/Library/Application Support/CorridorKey/`. Creates
  venv (default symlink mode — NOT `--copies`, that breaks
  `@rpath/libpython3.12.dylib` lookup), pip-installs runtime deps,
  finds the highest AE version via `/Applications/Adobe After Effects *`
  glob + trailing-integer extraction, copies plugin bundle into the
  detected `Plug-Ins/Effects/`, re-codesigns ad-hoc with
  `codesign --force --sign - --timestamp=none`, writes `VERSION` file.
  Logs to `/Library/Logs/CorridorKey-install.log`.
- `installer/macos/requirements.txt` — deps the postinstall pip-installs:
  corridorkey-mlx (git), numpy, Pillow, opencv-python-headless, msgpack.
- `plugin/src/CorridorKeyAE_Bridge.cpp` — `FindRepoRoot()` now includes
  `/Library/Application Support/CorridorKey` as a macOS candidate so
  the bridge finds the runtime delivered by the `.pkg`.
- CI: `installer-macos` job in `.github/workflows/ci.yml` runs on
  `macos-latest`, depends on `plugin-build`, downloads the
  `CorridorKey-macOS` artifact (which is a zip of the `.plugin` bundle —
  `actions/upload-artifact@v4` flattens single-dir parents, so we
  zip-wrap before upload), unpacks into `build/plugin/`, runs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iamjoshuadavies/corridorkey-ae](https://github.com/iamjoshuadavies/corridorkey-ae) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
