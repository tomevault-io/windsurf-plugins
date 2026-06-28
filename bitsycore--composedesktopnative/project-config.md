---
trigger: always_on
description: Guidance for Claude Code working in this repository. This is the only
---

# CLAUDE.md

Guidance for Claude Code working in this repository. This is the only
context the next agent will have — read it first.

## Project Overview

**ComposeNativeSDL3** — a Kotlin/Native subset of Compose Desktop running
on SDL3. No JVM. Compiles to native binaries for macOS (arm64), Linux
(x64/arm64), and Windows (mingwX64).

The project re-implements just enough of `androidx.compose.*` to host
Composable hierarchies, measure/place them as a layout tree, dispatch
SDL3 input events into the tree, and render them. Rendering is pluggable:

- **Skia** (via Skiko klibs) on macOS + Linux — Metal / OpenGL / CPU raster
- **SDL3** (SDL3_ttf + `SDL_RenderGeometry`) on Windows, and on
  macOS/Linux when `-Prenderer=sdl3` is set

## Module Layout

Local module names drop the `compose-desktop-native-` / `compose-desktop-`
prefix for terseness; publication artifact IDs add it back (so consumers see
`compose-desktop-native-core` etc. on the dependency line). All modules share
the `com.compose.desktop.native` Kotlin package; the re-implemented Compose
APIs keep their upstream `androidx.compose.*` names, in `core/commonMain`.

- `core/` (publishes as `compose-desktop-native-core`) — renderer-agnostic
  base: the `androidx.compose.foundation` / `.ui` / `.animation` re-impl,
  `RenderBackend` interface, `GpuMode`, `SDL3Backend`, window / clipboard /
  event / resource IO, and the bundled default font. Owns the `sdl3`
  cinterop. **No Material code** — Material widgets live in `:material`.
- `material/` (publishes as `compose-desktop-native-material`) — Material
  widgets re-implemented on top of `:core` (Button / Text / MaterialTheme /
  Surface / TextField / Slider / Switch / Checkbox / Radio / Chip / Card /
  Dialog / DropdownMenu / SegmentedButton / Snackbar / Tooltip /
  ProgressIndicator). Apps that only want the foundation+ui base without
  Material can skip pulling this in.
- `renderer-sdl3/` (publishes as `compose-desktop-native-renderer-sdl3`) —
  pure-SDL3 renderer (+ `sdl3_ttf`, `sdl3_image`, `freetype` cinterops).
  Exposes `createRenderBackend(...)` / `rendererPreferredGpuMode()`. All
  four native targets.
- `renderer-skia/` (publishes as `compose-desktop-native-renderer-skia`) —
  Skia/Skiko renderer (Metal / OpenGL / CPU bridges). Same two functions.
  **macOS + Linux only** — Skiko publishes no mingwX64 artifact.
- `window/` (publishes as `compose-desktop-native`) — what apps depend on.
  Owns `nativeComposeWindow()` and selects a renderer per target by depending on
  exactly one renderer module: mingwX64 → sdl3 (always); macOS/Linux →
  skia, or sdl3 under `-Prenderer=sdl3`. Re-exports `:core` + `:material`
  via `api`.
- `material-symbols/{outlined,rounded,sharp}/` (publishes as
  `compose-desktop-material-symbols-{outlined,rounded,sharp}`) — Material
  Symbols icon-font modules. Each ships its variable font (downloaded at
  build time from Google) and a single `MaterialSymbols{Style}` object
  with a `@Composable operator fun invoke(...)` that auto-installs the
  font on first call.
- `demo/` — flagship example app: a full showcase of the re-implemented
  Compose + Material widgets and features (30+ sidebar screens; `--gpu` /
  `--screen` / `--screenshot` CLI). Depends on `:window` + the three
  `material-symbols` styles. Not published (app only).
- `apidemo/` — a Postman-style **API Manager** built on the library: HTTP
  request collections (packs / nested sub-packs / linked copies), a
  Session → Pack → Request inheritance ladder for variables / headers /
  query-params / client-certs (innermost wins, with per-level overrides),
  syntax-highlighted JSON/XML/YAML/HTML body editors, a response viewer with
  TLS-chain inspection, mTLS client certs, drag-and-drop tree, request history
  and file-based sessions. Networking is Ktor's Curl engine (one bundled
  static libcurl per target — Schannel on Windows, OpenSSL on macOS/Linux) +
  kotlinx.serialization + okio. Not published (app only).

### How renderer selection works

Both renderer modules expose identically-signed `createRenderBackend` /
`rendererPreferredGpuMode` in the same package. `:window` has a thin
per-target `expect`/`actual` (`makeRenderBackend` / `preferredGpuMode`,
in `RenderBackendFactory.{kt,mingw.kt,macos.kt,linux.kt}`) whose actuals just
forward to those functions — and since the build links **exactly one** renderer
module per target, the call resolves unambiguously ("include one" selection).
No conditional `srcDir`s; each renderer module has its own per-OS source sets
(`mingwMain` / `macosArm64Main` / `linuxMain`) for `rendererPreferredGpuMode`.
`SDL3Backend` only exposes `COpaquePointer` (never `sdl3.*` types), so each
module declares its own `sdl3` cinterop and reinterprets — no cross-module
cinterop export needed.

### Key files (start here)

- `window/src/nativeMain/.../ComposeWindow.kt` — main loop, recomposer
  lifecycle, event dispatch; calls the per-target makeRenderBackend.
- `core/src/nativeMain/.../ComposeNativeWindow.kt` — per-window handle
  (title / size / fullscreen / rendererName / close), CompositionLocal + scope.
- `core/src/nativeMain/.../RenderBackend.kt` — the interface.
- `core/src/nativeMain/.../GpuMode.kt` — the sealed renderer/driver picker.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitsycore/ComposeDesktopNative](https://github.com/bitsycore/ComposeDesktopNative) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
