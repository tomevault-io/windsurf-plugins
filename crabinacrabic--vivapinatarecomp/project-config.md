---
trigger: always_on
description: This file is written for AI coding agents (Claude Code, Codex, Cursor, …). A person has probably given you the link to this repository and asked you to install the game, or to work on the code.
---

# AGENTS.md — instructions for AI agents

This file is written for AI coding agents (Claude Code, Codex, Cursor, …). A person has probably given you the link to this repository and asked you to install the game, or to work on the code.

- **Installing the game for a player** → section 1.
- **Changing the code of this project** → section 2.

Talk to the user in their own language. Many users of this project speak Russian.

---

## 1. Install Viva Piñata Recomp for a player (Windows)

**Goal:** build `out\build\local-win-relwithdebinfo\vivapinata.exe` from this repository and the user's own game disc image, then start it through `run_game.bat`.

### 1.0 Ground rules

- **Never download the game** or any game files from the internet. The game must come from the user's own disc image; ask for the path to their `.iso`.
- Ask before you start installers (Visual Studio, Git, Python, 7-Zip) and before accepting licences.
- The project path must contain **only ASCII characters**, for example `C:\Games\VivaPinataRecomp`. The ReXGlue code generator crashes with `0xC0000409` on paths with non-ASCII characters.
- Do not commit or redistribute `game_files\`, `game_files\Beta\bundles\russian.bnl` or any files from the ZoG translation.
- Visual Studio (`F7`) is the build path the maintainer tests. The command-line build in step 1.5 B should work, but it is not part of the maintainer's routine.

### 1.1 Check the machine

- Windows 10/11 x64, a GPU with Direct3D 12 (`dxdiag`), about 25 GB free: Visual Studio ~12 GB, game files ~5 GB, the ISO ~8 GB (it can be deleted after unpacking).

### 1.2 Install the tools

| Tool | How | Notes |
| :-- | :-- | :-- |
| Git | `winget install -e --id Git.Git` | |
| Visual Studio 2026 (v18) Community | `winget search Microsoft.VisualStudio`, take the 2026 Community package, then add `--override "--passive --wait --add Microsoft.VisualStudio.Workload.NativeDesktop --add Microsoft.VisualStudio.Component.VC.Llvm.Clang --add Microsoft.VisualStudio.Component.VC.Llvm.ClangToolset --includeRecommended"` | Needs the C++ workload, clang 22 (the *C++ Clang tools for Windows* component), CMake and Ninja (come with the workload). VS 2022 is untested. |
| extract-xiso | Download `extract-xiso-Win64_Release.zip` from https://github.com/XboxDev/extract-xiso/releases/latest and unzip it | Unpacks the Xbox 360 ISO |
| Python 3 | `winget install -e --id Python.Python.3.12` | Only for the Russian language and the dev tools |
| 7-Zip | `winget install -e --id 7zip.7zip` | Only for the Russian language |

### 1.3 Get the project

```powershell
git clone https://github.com/crabinacrabic/VivaPinataRecomp.git C:\Games\VivaPinataRecomp
```

### 1.4 Check and unpack the game (before the first build)

The ISO must be **Viva Pinata (USA, Europe) (En,Ja,Fr,De,Es,It,Nl,Pt,Sv,No,Zh,Ko,Pl,Cs,Hu,Sk)**, Title ID `4D5307F2`, version `0.0.0.1`.

```powershell
(Get-FileHash "C:\path\to\game.iso" -Algorithm MD5).Hash      # expected 3902321DFE15D7D2510A96114DBA625A
extract-xiso -x -d C:\Games\VivaPinataRecomp\game_files "C:\path\to\game.iso"
(Get-FileHash C:\Games\VivaPinataRecomp\game_files\default.xex -Algorithm SHA1).Hash
                                                              # expected 130DBBE05328EECA23AB830BC8ED3337064EDDD3
Test-Path C:\Games\VivaPinataRecomp\game_files\Beta\bundles\englishus.bnl   # expected True
```

- `default.xex` and `Beta\` must be directly inside `game_files\`. If extract-xiso created a subfolder, move its contents up.
- A different ISO hash is only a warning. A different `default.xex` SHA-1 means the recompiled code will not match: tell the user this edition is not supported.
- Unpack **before** the first CMake configure: configuring runs the code generator on `game_files\default.xex`.

### 1.5 Build

On the first configure, CMake downloads ReXGlue SDK `0.10.0.8-dev.g1406e1b` into `rexglue\win-amd64\` (`cmake/fetch-rexglue-sdk.cmake`, a GitHub release `nightly-20260915-1406e1b7`). Then it runs `rexglue codegen vivapinata_manifest.toml` once, which writes `generated\default\`. The first build compiles about 100 generated C++ files. Expect 10–40 minutes in total.

**A. Visual Studio (tested; ask the user to do this):**
1. Open Visual Studio 2026 → **File → Open → Folder…** → `C:\Games\VivaPinataRecomp`.
2. Wait for "CMake generation finished" in the Output window.
3. In the toolbar, select the configuration **`local-win-relwithdebinfo`**.
4. **Build → Build All** (`F7`).

**B. Command line (only when the user asks you to build without the IDE):**

```powershell
$vs = & "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe" -latest -products * `
      -requires Microsoft.VisualStudio.Component.VC.Llvm.Clang -property installationPath
Import-Module "$vs\Common7\Tools\Microsoft.VisualStudio.DevShell.dll"
Enter-VsDevShell -VsInstallPath $vs -SkipAutomaticLocation -DevCmdArguments "-arch=x64 -host_arch=x64"
$env:PATH = "$vs\VC\Tools\Llvm\x64\bin;$env:PATH"     # the presets use clang / clang++ from PATH
Set-Location C:\Games\VivaPinataRecomp
cmake --preset local-win-relwithdebinfo
cmake --build out/build/local-win-relwithdebinfo
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crabinacrabic/VivaPinataRecomp](https://github.com/crabinacrabic/VivaPinataRecomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
