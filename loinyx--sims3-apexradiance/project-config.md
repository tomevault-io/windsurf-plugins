---
trigger: always_on
description: **Apex Radiance** ("Apex Radiance for The Sims 3"; file `ApexRadiance.asi`; author @loinyx; renamed 2026-09-28 from
---

# CLAUDE.md: Apex Radiance

## What this is
**Apex Radiance** ("Apex Radiance for The Sims 3"; file `ApexRadiance.asi`; author @loinyx; renamed 2026-09-28 from
"Sims3 Settings Setter Apex Edition" / "S3SS Apex" / `S3SSApex.asi`) is a native mod for The Sims 3 (Steam 1.67.2,
`TS3W.exe`, 32-bit). It is an ASI loaded by Ultimate ASI Loader, running next to an unmodified official
Sims3SettingsSetter. It hooks the D3D9 device (the game runs on the official DXVK 3.1.1 `d3d9.dll`) and patches game
code in memory: Detours, pattern scans, ImGui menu, TOML config. Visible names come from `apex_version.h`
(`APEX_PRODUCT_NAME`, `APEX_PRODUCT_TAGLINE`); internal identifiers keep "Apex" (namespaces, `ApexPatch`, `APEX_`
macros, `apex_*` source files).

Features: Night Lighting (rebuilt night lamp light on ground, roads, floors, walls, roofs, water, foliage, objects,
fences, snow; key `NightTerrainRelight`), Every-Story Ground Light (`SplitLevelGroundLight`, lot lamps on any story
light the ground; part of Night Lighting), Reflections, Picture filters (SDR), Edge Smoothing
(SMAA/FXAA), Depth Blur, Borderless window, Performance (Faster Game File Lookups `ResourceLookupCache`, off by default
until tested, with Remember Missing Files `ResourceLookupMisses` (negative entries + write epochs) and Faster File Lists
`FileListCache` (GetKeyList cache), both off by default until tested; Lot Lighting While Moving `LotLightingMotion`;
Wall Shading While Moving `WallShadingWhileMoving` (defers the wall AO pass while moving, on by default); Faster Texture
Compression `FastTextureCompression`, a
bit-identical rewrite of the game's CPU DXT encoders, and Faster Cache Compression `FastCacheCompression`, a faster
RefPack compressor in the game's format, both off by default until tested; Spread New Objects Over Frames
`SceneNodeBudget`, a budgeted copy of the scene's pending-node drain while the camera moves, and Faster Object Lookups
`ObjectLookupIndex`, a validated index for the object/lot lookup by ID, both experimental and off by default; offline
tests in `tools\dxt_test` and `tools\refpack_test`; `docs/features/performance.md`), Frame Profiler (dev build only), plus dev tools (Light Probe
Ctrl+Shift+F7, Light Diag Ctrl+Shift+F8, Frame Capture Ctrl+Shift+F9, Lot Map Probe, census). Menu: Violet UI
(sidebar plus feature cards), hotkey Ctrl+Shift+F11. Smooth Streaming, Script GC Scheduler and Service Frame Budget
were removed (see below).

**Scope (user decision 2026-09-28):** HDR output, Native HDR, Ambient Occlusion, Smooth Streaming, Script GC Scheduler
and Service Frame Budget are removed from the standalone. Their findings are kept in `docs/removed-features.md`; do not
bring them back without the user asking.

## State of the code
- Frozen combined build (Apex inside a fork of S3SS): `%USERPROFILE%\Desktop\S3SS-dev\Sims3SettingsSetter\`, branch
  `night-remake`, tag `combined-final` (commit 45e36e2, local only). Full copy in
  `Backups Sims 3\16-antes da separacao (codigo completo)`. Treat it as read-only reference.
- This folder (`S3SSApex\`; the folder name is not changed yet, the user decides) is Apex Radiance, the standalone ASI
  that runs next to an unmodified official S3SS. The plan is `%USERPROFILE%\Desktop\S3SS-dev\PLANO-SEPARACAO.md`,
  summarised in `docs/architecture.md`. New work goes here. The framework is rewritten from scratch (no S3SS code).
- The docs cite files by their combined-tree names; the standalone keeps the module names.

## Read before touching anything
- `docs/README.md`: index. Then `docs/architecture.md` and `docs/workflow.md`.
- Before any lighting change: `docs/features/night-lighting/README.md`, the sub-part doc, and the engine docs
  (`docs/engine/`). Each feature doc has a "Pitfalls and failed approaches" section. Do not retry what is listed there
  without new evidence.
- Raw sources, in Portuguese and chronological (later entries win): `S3SS-dev\NOTAS-ILUMINACAO.md`, `PASSO3-PLANO.md`,
  `ROADMAP-NIGHT-REMAKE.md`. Decompile: `S3SS-dev\re\out`. Game shaders: `Game\Bin\Shaders_Win32.precomp` (read-only).

## Build
MSBuild: `C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\MSBuild\Current\Bin\MSBuild.exe`, v143,
Release|Win32, C++20, static CRT, vcpkg triplet `x86-windows-static` (always `/p:VcpkgEnableManifest=false`).

Apex Radiance (this folder; `ApexRadiance.sln` / `ApexRadiance.vcxproj`, `TargetName` `ApexRadiance`). The user
compiles; do not build unless asked:
```
MSBuild ApexRadiance.sln /p:Configuration=Release /p:Platform=x86 /p:VcpkgEnableManifest=false                     -> Release\ApexRadiance.asi (dev)
MSBuild ApexRadiance.sln /p:Configuration=Release /p:Platform=x86 /p:VcpkgEnableManifest=false /p:ApexPublic=true  -> Public\ApexRadiance.asi (public)
```
`/p:ApexPublic=true` defines `S3SS_PUBLIC` (objects in `Public\obj\`). Combined tree (frozen, for reference):
`MSBuild Sims3SettingsSetter.sln ... [/p:S3SSPublic=true]` -> `Release\` / `Public\S3SSApex.asi`.

Flavours (`build_flavor.h`): dev = everything plus dev tools and "Developer" UI sections; public = `S3SS_PUBLIC` /
`kPublicBuild`, dev tools compiled out. The user plays the dev build; releases ship the public build.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loinyx/Sims3-ApexRadiance](https://github.com/loinyx/Sims3-ApexRadiance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
