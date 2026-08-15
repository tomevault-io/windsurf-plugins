---
trigger: always_on
description: Guidance for Claude Code working in this repository. This is the primary
---

# CLAUDE.md

Guidance for Claude Code working in this repository. This is the primary
context - read it first, then look at the files it points to.

## Documentation map

- [README.md](README.md) - public overview + quickstart (bridge plugin,
  `nativeComposeWindow`, sample apps).
- Renderer - one Skia `RenderBackend` (Metal on macOS / OpenGL on Linux+Windows /
  CPU-raster fallback) driving upstream's vendored GraphicsLayer + Canvas engine.
  Retained per-node display lists: transform / alpha / clip changes REPLAY without
  re-recording (only a content change or resize re-records; dirty-region rendering
  is a non-goal). Text is a reduced-local port over Skia `skparagraph`. Read these
  before touching renderer / graphics-actual / layer-engine code:
  `SkiaRenderBackend.kt`, `RenderBackend.kt`, `GpuMode.kt`, `ComposeRootHost.kt`
  (root host + snapshot-observation sweep - omitting `clearInvalidObservations()`
  once leaked the whole graph), `ComposeOwner.kt`, and `SkiaParagraph.native.kt` /
  `SkiaParagraphEngine.kt`. Open renderer/fidelity work lives in [PLAN.md](PLAN.md).
- [TOOLING.md](TOOLING.md) - build/vendor/verify scripts and workflows
  (build-sdl, sync + drift checks, parity, probe, profiler, coverage,
  verify-mac) + the version map and the ref-bump / release runbooks.
- [PLAN.md](PLAN.md) §2 - audited list of no-ops, stubs, and hardcodes left in the
  port, with P0/P1/P2 severity, plus the road-to-1.0.0 fidelity / SDL / release work
  (this subsumes the former TODO.md, which was never committed).
- This file - architecture, module layout, vendoring rules, source-set
  hierarchy, density flow, conventions, and common pitfalls.

## What this project is

**ComposeNativeSDL3** - a Kotlin/Native port of Compose Multiplatform running
on SDL3, no JVM. Compiles to native binaries for macOS (arm64), Linux
(x64/arm64), Windows (mingwX64).

Rendering is **Skia everywhere** behind one `RenderBackend` - Metal / OpenGL
/ CPU raster:

- **macOS + Linux** link the OFFICIAL Skiko klibs from Maven.
- **Windows (mingwX64)** links the bitsycore skiko FORK - skiko+Skia compiled
  into `skiko-windows-x64.dll` with a flat extern-C surface, bound from K/N via
  an embedded GNU import lib, published to GitHub Packages as
  `com.bitsycore.skiko:skiko:0.150.1-mingw.1` (override with
  `-PskikoMingwVersion`). The runtime DLL is auto-provisioned next to the exe by
  the bridge plugin (`installWindowsSkiaDll`).

Windowing, input, audio, filesystem access, and the OS-integration surface
(file dialogs, clipboard, "open in Finder/Explorer"…) all go through
**SDL3**. The runtime (`androidx.compose.runtime.*`: composition, snapshots,
recomposer, `mutableStateOf`, `remember`, …) is the **official
`org.jetbrains.compose.runtime` klibs from Maven** - this project only
re-implements the layers on top (`androidx.compose.ui.*`, `.foundation.*`,
`.animation.*`, `.material3.*`).

## Module layout

Library modules mirror upstream Compose Multiplatform's `compose/` tree.
The SDL layer is two modules: `:sdl-core` (the NAKED sdl3 cinterop + platform
primitives - zero Compose dep, like `skiko`) at `sdl/sdl-core/`, and
`:desktop-native-window` (the SDL3 main-loop shell + app entry point) at
`compose/desktop/native/window/`. `:ui` depends on `:sdl-core` and its renderer
+ SDL↔Compose bridges pick the cinterop from it.

One Gradle module per upstream artifact; the directory mirrors the upstream
`compose/` path, the gradle path is kept short (redirected via `projectDir`).

```
compose/
├── ui/
│   ├── ui/                          → :ui        - androidx.compose.ui.* CORE (Modifier, LayoutNode,
│   │                                               composition, semantics, input, focus) + com.compose.sdl.* -
│   │                                               the Skia RenderBackend + GPU bridges + the SDL↔Compose
│   │                                               bridges (events / clipboard / cursors / window). Depends on
│   │                                               :ui-graphics, :ui-text, :sdl-core. (ui-graphics + ui-text +
│   │                                               the sdl3 cinterop were split OUT - upstream layout.)
│   ├── ui-graphics/                 → :ui-graphics - androidx.compose.ui.graphics.* + the Skia actuals
│   │                                               (SkiaBackedCanvas/Path/Paint, GraphicsLayer, SkiaImageCache,
│   │                                               painter/image + resource seams). → skiko; SDL-free.
│   ├── ui-text/                     → :ui-text   - androidx.compose.ui.text.* + the skiko text engine
│   │                                               (SkiaParagraph → NativeParagraphOps → SkiaFonts, IconFont,
│   │                                               NamedFont). → :ui-graphics, skiko; SDL-free.
│   ├── ui-util/                     → :ui-util       - androidx.compose.ui.util.* (+ Experimental/InternalComposeUiApi)
│   ├── ui-geometry/                 → :ui-geometry   - androidx.compose.ui.geometry.*
│   ├── ui-unit/                     → :ui-unit       - androidx.compose.ui.unit.*
│   ├── ui-backhandler/              → :ui-backhandler - androidx.compose.ui.backhandler.*
│   └── ui-tooling-preview/          → :ui-tooling-preview - androidx.compose.ui.tooling.preview.*

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitsycore/compose-desktop-native](https://github.com/bitsycore/compose-desktop-native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
