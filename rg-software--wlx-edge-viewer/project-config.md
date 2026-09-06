---
trigger: always_on
description: > **Repository hosting**: This project is on **GitHub** at `rg-software/wlx-edge-viewer`. Issues and pull requests are read/written through the **GitHub MCP** (`owner=rg-software`, `repo=wlx-edge-viewer`) — do **not** use the Gitea MCP for this repo. The remote `git` origin also points at GitHub.
---

# AGENTS.md

> **Repository hosting**: This project is on **GitHub** at `rg-software/wlx-edge-viewer`. Issues and pull requests are read/written through the **GitHub MCP** (`owner=rg-software`, `repo=wlx-edge-viewer`) — do **not** use the Gitea MCP for this repo. The remote `git` origin also points at GitHub.

Lister plugin (32/64-bit Windows via Total Commander; 64-bit Linux via Double Commander) that renders Markdown, AsciiDoc, RST, HTML, MHT, images, directories and PDF through a WebView2 / Qt Web Engine backend. One C++23 source tree; the platform-specific parts are isolated in sibling files (`WebView2Backend.{h,cpp}`, `QtWebEngineBackend.{h,cpp}`, `Platform_Win.cpp`, `Platform_Linux.cpp`, `EdgeLister_Win.cpp`, `EdgeLister_Linux.cpp`, `DirProcessor_Win.cpp`). Build system differs per platform: MSBuild + vcpkg (Windows) vs CMake + Qt6 (Linux).

## Build

### Windows

- Requires MS Visual Studio 2022 (v143 toolset) + vcpkg with MSBuild integration. Deps are pinned in `vcpkg.json` (manifest mode): `webview2`, `wil`, `catch2`, `pulzed-mini` (the mINI ini parser, header-only, version locked by the existing `builtin-baseline` at 0.9.14 — identical content to the former vendored copy), static triplets `x86-windows-static`/`x64-windows-static` (set in `EdgeViewer.vcxproj`). `vcpkg_installed/` holds the built triplets (gitignored).
- `msbuild` is not on PATH: run from the "MSVS Developer Command Prompt" (of the VS 2022 install, so `%VCINSTALLDIR%` points at `...\2022\Community`), or as `BuildMakeSetup.bat` does — `vcvarsall.bat x86|x64` then `msbuild ... /p:UseEnv=true`.
- MSBuild vcpkg integration discovers the root via `%VCPKG_ROOT%` (set the env var to your vcpkg clone, e.g. `C:\vcpkg`). Manifest mode + static triplets; deps rebuild automatically against the active toolset.
- Release packaging: `BuildMakeSetup.bat` builds Release for both platforms, assembles `winbuild\Release\` (Resources + `EdgeViewer.wlx` = Win32 DLL renamed, `EdgeViewer.wlx64` = x64 DLL), zips to `Release-YYYYMMDD-Win.zip`.
- Per-config outputs land in `winbuild\EdgeViewer_<Platform>_<Config>\` (gitignored): Release DLLs are `EdgeViewer-Win32.dll`/`EdgeViewer-x64.dll`, Debug are `EdgeViewerD-...`.
- Tests (Windows): `msbuild EdgeViewer.Tests/EdgeViewer.Tests.vcxproj /p:Configuration=Release /p:Platform=x64` produces `winbuild\EdgeViewer.Tests_x64_Release\EdgeViewer.Tests.exe` (and same for Win32). 60 test cases / 263 assertions pass on both Windows platforms.

### Linux

- Linux builds via CMake plus Qt6 development packages: `qt6-base-dev`, `qt6-webengine-dev` (Debian/Ubuntu) or the `qt6-qtbase-devel` + `qt6-qtwebengine-devel` equivalents (Fedora/Arch), plus `pkg-config` and `cmake`.
  ```bash
  cmake -B build -S .
  cmake --build build -j
  cmake --install build --prefix ~/.local
  ```
  Output: `build/EdgeViewer.wlx64`. Install rules lay out the `.wlx64` with `assets/` and `edgeviewer.ini` next to it (`~/.local/share/doublecmd/plugins/edgeviewer/`) — `ProcessorInterface::assetsPath()` is `GetModulePath()/assets`, matching the Windows package layout where `BuildMakeSetup.bat` flattens `Resources\` contents beside the DLL.
- No Linux test suite (manual DC verification per OpenSpec task 6).
- `vcpkg.json` is unchanged on Windows; Linux uses system pkg-config (no vcpkg equivalent).
- No vendored mINI copy exists in the tree on either platform. Linux fetches the header at CMake configure time from upstream commit `a1ff72e` (`MINI_COMMIT` in `CMakeLists.txt`, `file(DOWNLOAD)` to `${CMAKE_CURRENT_BINARY_DIR}/mini/mini/ini.h`, `EXPECTED_HASH` pinned). A future mINI bump MUST move both pin sites in lockstep: the vcpkg `builtin-baseline` (Windows) and `MINI_COMMIT` (Linux).
- Linux exports are controlled by a GNU ld version script (`CMakeLists.txt` builds `EdgeViewer.version`: `global: ListLoadW; ...; local: *;`) with the 12 WLX symbols declared `extern "C"` in `DllMain.cpp`. Do **not** combine this with `-fvisibility=hidden`: empirically, hidden-visibility symbols are NOT exported by GNU ld even when listed in the version script, so the CMake visibility presets were removed. `EdgeViewer.wlx64` exposes the same WLX symbols Windows does (`ListLoadW`, `ListLoadNextW`, etc.).

### Branching

- `master` — the single source of truth. Windows and Linux live in one tree; no separate port branch.
- Tag `windows-refactor-stable` anchors the Windows baseline at `5e44484` (clean cross-platform refactor before any Linux code). It is historical only.
- The former `port-to-double-commander-linux` branch (Section 2 refactor + pre-fetch + Section 4 Linux backend) was merged into `master` and no longer exists. Its orphaned history and the dangling pre-rewrite commit `31b9e79` are referenced only for archaeology; do not treat them as ancestors.

## Architecture

The canonical architecture/conventions reference is `openspec/config.yaml` → `context`. It is loaded into every AI-assisted session automatically (local tooling, not part of the repo), so there is a single source of truth — do not duplicate it here.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rg-software/wlx-edge-viewer](https://github.com/rg-software/wlx-edge-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
