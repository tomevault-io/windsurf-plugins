---
trigger: always_on
description: handles backdrop clip and progressive mask. Atlas grouping is not implemented
---

# Repository Guidelines

## Project Structure
- `app/` is the Android demo and manual verification surface.
- `imla/` is the Imla rendering library. Public Compose APIs, internal GL
  renderers, shader adapters, and renderer tests live here.
- `benchmark/` runs macrobenchmarks against `:app` through its `benchmark`
  build type.
- `demo/` contains static README artifacts. Update it only when shipped visuals
  change.
- Project-wide Gradle configuration lives in `settings.gradle.kts`,
  `build.gradle.kts`, `gradle/`, and `gradle/libs.versions.toml`.

## Common Source Locations
- Public Compose APIs live in root package
  `imla/src/main/java/dev/serhiiyaremych/imla/`. Start here for `ImlaHost`,
  `Modifier.effectGroup()`, `Modifier.effectLayer { ... }`,
  `EffectLayerScope`, and `EffectLayerBoundsProvider`.
- Renderer code starts in
  `imla/src/main/java/dev/serhiiyaremych/imla/internal/render/SceneRenderer.kt`.
- Effect group/layer modifiers live under
  `imla/src/main/java/dev/serhiiyaremych/imla/internal/modifier/`. Capture
  helpers live under `internal/capture/`.
- Layer internals live under
  `imla/src/main/java/dev/serhiiyaremych/imla/internal/layer/`. Start here for
  immutable snapshots, registry, resources, and layer geometry.
- GL owner, render target, pipeline, scheduler, renderer primitives, and shader
  adapters live under
  `imla/src/main/java/dev/serhiiyaremych/imla/internal/render/`.
- Metrics overlay support lives under `internal/metrics/`.
- Imla GL primitives live under
  `imla/src/main/java/dev/serhiiyaremych/imla/internal/render/`. Start here for
  `CoordinateOrigin`, framebuffers, textures, shader programs, GL state, and
  quad rendering utilities.
- AndroidX Compose source checkout:
  `/Users/syaremych/dev/projects/androidx/compose/ui/`.
  Use this when verifying Compose modifier, draw, layer, graphics-layer,
  `AndroidExternalSurface`, coordinate, semantics, or lifecycle behavior.
- AndroidX Compose UI common node/draw code:
  `/Users/syaremych/dev/projects/androidx/compose/ui/ui/src/commonMain/kotlin/androidx/compose/ui/`.
  Useful anchors include `draw/`, `graphics/GraphicsLayerModifier.kt`,
  `node/DrawModifierNode.kt`, `node/LayoutNodeDrawScope.kt`,
  `node/NodeCoordinator.kt`, and `node/OwnedLayer.kt`.
- AndroidX Compose UI Android platform code:
  `/Users/syaremych/dev/projects/androidx/compose/ui/ui/src/androidMain/kotlin/androidx/compose/ui/`.
  Useful anchors include `platform/AndroidComposeView.android.kt`,
  `platform/GraphicsLayerOwnerLayer.android.kt`,
  `platform/RenderNodeLayer.android.kt`, `platform/ViewLayer.android.kt`, and
  `layout/GraphicLayerInfo.android.kt`.
- AndroidX Compose graphics-layer implementation:
  `/Users/syaremych/dev/projects/androidx/compose/ui/ui-graphics/src/`.
  Useful anchors include
  `commonMain/kotlin/androidx/compose/ui/graphics/layer/GraphicsLayer.kt`,
  `androidMain/kotlin/androidx/compose/ui/graphics/layer/AndroidGraphicsLayer.android.kt`,
  `GraphicsLayerV23.android.kt`, `GraphicsLayerV29.android.kt`, and
  `LayerSnapshot.android.kt`.
- AndroidX graphics-core checkout:
  `/Users/syaremych/dev/projects/androidx/graphics/graphics-core/`.
  Use this when verifying GL renderer, EGL, HardwareBuffer, SyncFence,
  `CanvasBufferedRenderer`, or `SurfaceTextureRenderer` behavior.
- AndroidX graphics-core GL sources:
  `/Users/syaremych/dev/projects/androidx/graphics/graphics-core/src/main/java/androidx/graphics/opengl/`.
  Useful anchors include `GLRenderer.kt`, `GLThread.kt`,
  `FrameBufferRenderer.kt`, `FrameBufferPool.kt`, `FrameBuffer.kt`, and
  `QuadTextureRenderer.kt`.
- AndroidX graphics-core capture/import sources:
  `/Users/syaremych/dev/projects/androidx/graphics/graphics-core/src/main/java/androidx/graphics/`
  and
  `/Users/syaremych/dev/projects/androidx/graphics/graphics-core/src/main/java/androidx/hardware/`.
  Useful anchors include `CanvasBufferedRenderer.kt`,
  `CanvasBufferedRendererV29.kt`, `CanvasBufferedRendererV34.kt`,
  `SurfaceTextureRenderer.kt`, `HardwareBufferFormat.kt`,
  `HardwareBufferUsage.kt`, and `SyncFenceCompat.kt`.

## Commands
- `./gradlew -q compileDebugKotlin` is the default quick compile check.
- `./gradlew -q :imla:build` focuses on the library.
- `./gradlew -q build` compiles all modules, runs tests, and enforces lint.
- `./gradlew -q test` runs JVM tests.
- `./gradlew -q lint` runs Android lint with warnings as errors.
- `./gradlew -q :app:assembleDebug` builds the debug APK for manual device work.
- `./gradlew -q :app:connectedDebugAndroidTest` runs instrumentation tests on a
  connected device or emulator.
- Use `tools/adb-timeout` for every ADB command in this repo. Example:
  `tools/adb-timeout --timeout 10 devices -l`.

## Renderer Contract
- Active branch line: `syaremych/imla-2.0`.
- Public API is modifier-first: `ImlaHost`, `Modifier.effectGroup()`, and
  `Modifier.effectLayer { ... }`.
- Do not pass renderer or OpenGL objects into child slots. Slots register UI
  state through the internal layer registry exposed by the effect group and host
  scope.
- Do not reintroduce the removed renderer-bridge API: `rememberUiLayerRenderer`,
  `UiLayerRenderer`, `ImlaRenderPipeline`, `CopyLessRenderingPipeline`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [desugar-64/imla](https://github.com/desugar-64/imla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
