---
trigger: always_on
description: This is the repository-level entry point for agents. Use it when the user asks
---

# Agent guide: install and run WoW Forever Codex

This is the repository-level entry point for agents. Use it when the user asks
to install, run, update or troubleshoot the addon and companion. A request to
edit documentation alone does not authorize reinstalling or restarting anything.

Canonical repository: <https://github.com/0xInuarashi/wow-forever-codex>.
The product name and addon folder remain **Forever Bridge** and
**CodexPixelBridge**. Do not rename the addon folder to match the repository.

## What you are installing

- `addon/CodexPixelBridge/`: native WoW Lua UI and optical/font protocols.
- `companion/`: local Python GUI, screen capture, durable inbox and Codex adapter.
- `tools/install_addon.py`: the installer. It copies code and creates missing
  font/image assets without resetting existing resources.
- `Launch Companion.pyw`: optional desktop launcher; uses the repository's
  `state/launcher.json` and native Codex discovery.

The current production bank has **65,535 first-use fonts**. A clean Git checkout
contains no generated TTF/TGA files. Copying the addon source folder alone is
not a complete installation. The installer also creates 4,096 legacy image
placeholders and the fixed diagnostic resources; normal replies use fonts.

## Boundaries and existing installations

1. Use documented addon APIs, ordinary file operations and screen capture only.
   Never inject code into WoW, read game-process memory, bypass client checks,
   synthesize game input or use protected URL calls. The user types game commands.
2. Preserve all existing font/image assets, SavedVariables, inbox data, settings
   and capture calibration. Never reset a bank or its next-slot counter as a
   routine repair. Never write in place through a shared font hard link.
3. Keep one companion per inbox/addon. Reuse the running instance when possible.
   Before restarting your own companion, check that its queue is idle and close
   it normally. Startup marks unfinished inbox jobs interrupted; do not start a
   second process against that database. Do not terminate or restart WoW.
4. Keep the Codex sandbox `read-only` unless the user has authorized project
   edits. Send prompts as data through the existing adapter; do not interpolate
   prompt text into shell commands.
5. Complete authorized setup and local checks before asking for the final manual
   game step. Ask only for unresolved choices or required sign-in/game input.
   Explain the specific missing detail or reason for a reload.
6. Keep machine-specific paths, screenshots, credentials and runtime data out of
   commits. Observe `.gitignore`; generated fonts can contain actual reply bytes.

## 1. Resolve the installation paths

Discover or infer these from the existing installation before asking the user:

| Variable | Meaning |
| --- | --- |
| `bridgeRoot` | Stable local checkout containing `companion/`, `tools/` and the launcher |
| `bridgeClient` | Actual Forever client directory containing the game executable and `Interface/` |
| `bridgeAddon` | `bridgeClient/Interface/AddOns/CodexPixelBridge` |
| `bridgeProject` | Existing work directory Codex should operate in; separate from the game installation |
| `bridgeState` | Existing companion state directory; normally `bridgeRoot/state` |
| `bridgeCodex` | Verified native `codex.exe` path |

For upgrades, read the current companion's configuration and launch location
first. A source copy and the running companion may be in different folders.
Changing folders must not silently create a new inbox or orphan the desktop
shortcut. Do not print private inbox contents while discovering paths.

Windows, Python **3.12+ with tkinter**, and a filesystem supporting hard links
(normally NTFS) are required. The tested client was Forever **1.60.1.69913 / TOC
16001**, commonly under `_classic_beta_`. Check the actual installation instead
of assuming a drive letter, retail client, or another Classic folder. If WoW is
running, its executable path is useful discovery metadata; avoid printing its
full command line, which may contain login arguments. If several installations
are plausible and no saved path resolves the choice, ask which one to target.

All examples below run in PowerShell. Replace example paths with discovered
absolute paths. Do not overwrite a valid saved value with an example.

```powershell
$bridgeRoot = (Get-Location).Path  # Run from this checkout's root.
$bridgeClient = 'C:\path\to\World of Warcraft\_classic_beta_'
$bridgeAddon = Join-Path $bridgeClient 'Interface\AddOns\CodexPixelBridge'
$bridgeProject = 'C:\path\to\work-project'
$bridgeState = Join-Path $bridgeRoot 'state'
```

## 2. Prepare the Python runtime and locate Codex

Reuse a working project virtual environment. Otherwise, select an installed
Python 3.12+ and create one; substitute that interpreter for `python` below.
Do not overwrite an existing environment just because `python` on PATH differs.

```powershell
Set-Location -LiteralPath $bridgeRoot
python --version
# Run only if this checkout does not already have a usable .venv:
python -m venv .venv
$bridgePython = Join-Path $bridgeRoot '.venv\Scripts\python.exe'
& $bridgePython -m pip install -r requirements.txt

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xInuarashi/wow-forever-codex](https://github.com/0xInuarashi/wow-forever-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
