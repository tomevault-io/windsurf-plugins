---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

PascalRAL (Pascal REST API Lite) is an Object Pascal **component suite** for building and consuming REST APIs. It is a library installed into an IDE, not an application: there is no `main`, no runnable binary, and **no automated test suite**. It targets Delphi XE+ and Lazarus/FPC from a single shared source tree in `src/`, with IDE packages in `pkg/Delphi` (`.dpk`/`.dproj`) and `pkg/Lazarus` (`.lpk`).

Agent-oriented navigation docs already exist in `.agents/` (written in Portuguese): `AGENT_QUICKSTART.md` (which unit to open per goal), `PROJECT_MAP.md` (full file map), `TASK_PLAYBOOKS.md` (per-task read order), `SKILLS.md`. Prefer them over re-crawling `src/`.

## Build / verify

There is nothing to run as a test command. **Verification means compiling the packages**, and the only CI (`.github/workflows/changelog.yml`) just regenerates `CHANGELOG.md` — it does not build.

Package build order is authoritative in the group files; the runtime package must build first because everything else requires it:
- Delphi: `pkg/Delphi/PascalRALComponents.groupproj`
- Lazarus: `pkg/Lazarus/PascalRALGroup.lpg`

```powershell
# Delphi (from an rsvars.bat-initialized shell)
msbuild pkg\Delphi\PascalRALComponents.groupproj /t:Build /p:Config=Release

# single package
msbuild pkg\Delphi\PascalRAL.dproj /t:Build /p:Config=Release

# Lazarus/FPC
lazbuild pkg\Lazarus\pascalral.lpk
lazbuild --build-ide= pkg\Lazarus\pascalraldsgn.lpk   # design-time pkg requires an IDE rebuild
```

**`msbuild` fails on a workstation with many components installed** — `MSB6003: The specified task executable "dcc" could not be run`. The cause is `DelphiLibraryPath` (the IDE's global library path, read from the registry): `CodeGear.Delphi.Targets` folds it into `-U`, `-R`, `-I` **and** `-O`, so a 12 KB library path becomes ~48 KB of command line. Nothing is wrong with the package.

It can be driven from `msbuild` anyway — trim that one property and turn package linking back on. This recipe builds every package correctly on such a machine:

```powershell
# BDS lib for the target platform + the .dcp store is all the compiler needs
$dlp = "$env:BDSLIB\Win32\release;$env:BDSCOMMONDIR\Dcp"

msbuild pkg\Delphi\Engine\IndyRAL.dproj /t:Build /p:Config=Release /p:Platform=Win32 `
  /p:DelphiLibraryPath="$dlp" /p:UsePackages=true `
  /p:DCC_UsePackage="rtl;IndySystem;IndyProtocols;IndyCore;PascalRAL;PascalRALDsgn"
```

Three traps, in the order they bite:

1. **`/p:UsePackages=true` is mandatory.** The targets emit `-LU` only `Condition="'$(UsePackages)'==true Or '$(DCC_EnabledPackages)'=='true'"`, and **no `.dproj` in this repo sets either**. Without it `msbuild` produces a package with Indy/FireDAC linked *statically* — it compiles clean and the IDE then refuses it with a duplicate-unit error. Watch the size: `IndyRAL.bpl` comes out at 1.5 MB instead of 45 KB, `RALDBFireDACLink.bpl` at 2.9 MB instead of 104 KB.
2. **Filter `DCC_UsePackage` against the `.dcp` that actually exist.** These lists accumulate whatever was installed when the `.dproj` was last saved; `IndyRAL.dproj` still names `IndyCore160`/`IndySystem160`/`IndyProtocols160`. With `-LU` on, a name with no `.dcp` is a hard `E2202: Required package 'IndyCore160' not found`. Keep only the entries with a matching `.dcp` under the lib or `Dcp` directory.
3. **The `Base` PropertyGroup's `DCC_UnitSearchPath` does not get applied this way.** It only matters for `SynopseRAL`, because every other package names its units with explicit `in '..\..\src\...'` paths in the `.dpk` while the mORMot units are external. Pass them yourself, `$(mormot2)` expanded:
   `/p:DCC_UnitSearchPath="<src\base>;<src\utils>;<src\engine\synopse>;<m>;<m>\core;<m>\lib;<m>\crypt;<m>\net;<m>\db;<m>\rest;<m>\orm;<m>\soa;<m>\app;<m>\script;<m>\ui;<m>\tools;<m>\misc"`.
   `mormot2` is an **IDE** environment variable, so `msbuild` does not see it — pass `/p:mormot2=...` or set it in the shell.

Healthy sizes after a full rebuild (Win32/Release): `PascalRAL` 542 KB, `PascalRALDsgn` 80, `IndyRAL` 45, `NetHttpRAL` 32, `SynopseRAL` 4432 (mORMot is statically linked — it has no runtime package, so this one is meant to be large), `RALDBPackage` 122, `RALDBFireDACLink` 104, `RALDBFireDACObjects` 91, `RALWizard` 136, `RALZStdCompress` 51, `RALBSONStorage` 66.

If you would rather bypass `msbuild` entirely, `dcc32` still works:

**When calling `dcc32`/`dcc64` directly, you must replicate `DCC_UsePackage` yourself.** Each `.dproj` carries the list of runtime packages its units come from, but the `.dpk`'s `requires` clause does *not* repeat it (`IndyRAL.dpk` requires only `PascalRALDsgn`). Compiling the `.dpk` with `--no-config` ignores the `.dproj` entirely, so Indy and FireDAC get **statically linked into the .bpl** — it compiles clean, then the IDE refuses to load it with a duplicate-unit error against `IndyProtocols290`/`FireDAC290`. Read `<DCC_UsePackage>` out of the `.dproj` and pass it as `-LU`:

```bash
dcc32 --no-config -B -Q -NS"System;System.Win;Winapi;Vcl;Data;Data.Win;Xml;Web;Soap;Datasnap" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenSourceCommunityBrasil/PascalRAL](https://github.com/OpenSourceCommunityBrasil/PascalRAL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
