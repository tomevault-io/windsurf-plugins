---
trigger: always_on
description: FixedMathSharp-Unity is the Unity Package Manager host for the
---

# FixedMathSharp-Unity Agent Instructions

## Purpose

FixedMathSharp-Unity is the Unity Package Manager host for the
engine-agnostic FixedMathSharp core library. It ships two installable package
variants:

- `com.mrdav30.fixedmathsharp` - standard package with `MemoryPack` support.
- `com.mrdav30.fixedmathsharp.lean` - no-MemoryPack package, preferred for
  Burst AOT-oriented Unity projects.

The actual Git repository root is `Assets/Packages`, not the outer Unity
project root. The outer project exists so Unity can compile, sync, export, and
validate the packages.

The core deterministic math library is maintained in the separate
`FixedMathSharp` repository:

- Git URL: `https://github.com/mrdav30/FixedMathSharp.git`

If the current workspace also contains a local checkout of that core repo, use
its `AGENTS.md` and source files as the source of truth for `Fixed64`, Q32.32
math, coordinate conventions, serialization semantics, performance
expectations, and deterministic behavior. Do not assume a fixed host-specific
path; discover the checkout from the current workspace, git remotes, or user
context.

## Current Package Facts

- Package manifests require Unity `2022.3` or newer.
- This Unity host project currently uses Unity `6000.3.9f1` according to
  `../../ProjectSettings/ProjectVersion.txt`.
- Both packages include the precompiled FixedMathSharp plugin assets under
  `Plugins/`.
- The standard package carries the MemoryPack-related dependencies; the Lean
  package intentionally omits MemoryPack.

## Source Of Truth

When shared Unity-managed code changes, edit `Build/Base` first.

`Build/Base` is synchronized into both package variants by
`Build/Editor/FixedMathSharpPackageSync.cs`. Do not hand-edit synced copies in
`com.mrdav30.fixedmathsharp` or `com.mrdav30.fixedmathsharp.lean` unless you
are intentionally changing package-specific files outside the sync set.

The sync-managed paths are:

- `COPYRIGHT`
- `LICENSE`
- `NOTICE`
- `Editor/Utility`
- `Editor/Drawers`
- `Runtime/Attributes`
- `Runtime/Extensions`
- `Samples/FixedMathSharpDemo/Scripts`

## Unity Sample Authoring And Export Layout

Unity package samples use two folder shapes in this repo:

- `Samples/` is the local authoring mirror. It is visible in the Unity editor
  so sample scenes, scripts, and references can be edited normally.
- `Samples~/` is the distributable package copy used by Git/package-source
  installs. Package manifests should point sample entries at `Samples~/...`.

For package variants, `Samples/`, `Samples.meta`, and `Samples~.meta` are
ignored by package-local `.gitignore` files. Do not commit package-variant
`Samples/` folders or top-level `Samples~.meta` files.

Keep nested `.meta` files inside samples. They preserve scene, prefab, material,
asmdef, and script references when Unity imports samples into
`Assets/Samples/...`.

When shared sample scripts change, edit `Build/Base/Samples` first and run the
package sync/export flow. The sync step hydrates missing package-variant
`Samples/` folders from tracked `Samples~/` folders, then copies shared sample
scripts into the visible authoring mirror. The exporter overwrites `Samples~/`
from `Samples/` and explicitly excludes `Samples/` from the exported package.

Unity hides tilde folders from the asset database, so the stock
`.unitypackage` exporter should not be treated as the source of sample-package
coverage unless a custom archive/export path is added.

Package-specific files live directly in each package and are not copied from
`Build/Base`, including:

- `package.json`
- package `README.md`
- asmdef files
- `Plugins/` DLL, PDB, XML, and dependency assets
- sample scene assets

Keep these aligned whenever package shape, install guidance, package variants,
or exported assets change:

- Root `README.md`
- `.agents/AGENTS.md`
- `Build/Base`
- `Build/Editor/FixedMathSharpPackageSync.cs`
- `Build/Editor/FixedMathSharpUnityPackageExporter.cs`
- `com.mrdav30.fixedmathsharp/**`
- `com.mrdav30.fixedmathsharp.lean/**`

## Repository Map

| Path | Purpose |
| --- | --- |
| `Build/Base` | Shared managed Unity code copied into both package variants. |
| `Build/Editor` | Unity editor tooling for syncing and exporting packages. |
| `com.mrdav30.fixedmathsharp` | Standard UPM package with MemoryPack support. |
| `com.mrdav30.fixedmathsharp.lean` | Lean UPM package without MemoryPack. |
| `Build/Base/Runtime/Extensions` | Shared Unity adapter helpers for transforms, matrices, bounds, vectors, and quaternions. |
| `Build/Base/Runtime/Attributes` | Shared Unity-facing attributes such as fixed-number angle and vector rotation attributes. |
| `Build/Base/Editor/Drawers` | Shared Unity inspector drawers for FixedMathSharp types and Unity-facing attributes. |
| `Build/Base/Editor/Utility` | Shared editor helpers, including serialized-property reflection utilities. |
| `Build/Base/Samples/FixedMathSharpDemo` | Shared demo scripts copied into both package variants. |

Ignore generated Unity output when reasoning about package source:

- `../../Library/`
- `../../Temp/`
- `../../Obj/`
- `UnityPackageExports~`

## FixedMathSharp Core Rules Still Apply

Unity integration code is an adapter layer. It should not weaken the core
library's deterministic design.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrdav30/FixedMathSharp-Unity](https://github.com/mrdav30/FixedMathSharp-Unity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
