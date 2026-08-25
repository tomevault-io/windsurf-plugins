---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

WhatsApp-style photo editor delivered as **two published Android libraries** plus a thin demo app. Built with Jetpack Compose. Kotlin 2.0.21, AGP 8.13.0, compileSdk 36, minSdk 24, Java 11 toolchain, Compose BOM 2024.09.00.

- `:photoeditor-core` — the UI-agnostic engine: canvas composable, element model, gestures, undo/redo, state holder, export. **No material3 dependency** — keep it that way; Mode A consumers must not inherit Material. Gradle namespace `com.muzamil.photoeditor.core`.
- `:photoeditor-ui-default` — the optional built-in UI: tool panels, bottom nav, top bar, token-based theme. Depends on core via `api` (consumers get core types transitively). Namespace `com.muzamil.photoeditor.uidefault`. **Dependency direction is one-way (`ui-default → core`), never the reverse.**
- `:app` — demo consumer with three sample screens: `DefaultUiSample` (Mode B, built-in UI), `CustomizedUiSample` (Mode B with custom emojis/swatches + `colorsFromMaterialTheme()` bridge), and `CustomUiSample` (Mode A, hand-rolled toolbar over core only — keep its imports core-only; it exists to prove the core API is sufficient).

All three share Kotlin package root `com.muzamil.photoeditor.*` (fine — AGP namespaces differ, so R classes don't collide), but `internal` visibility does NOT cross module boundaries: ui-default can only use core's public API.

Publishing: both library modules apply `maven-publish` (release variant + sources jar). Project-level `group`/`version` are set in each module's build file so ui-default's generated POM declares core with the right GAV. `jitpack.yml` builds both via `publishToMavenLocal` on JDK 17. Validate locally with `./gradlew :photoeditor-core:publishToMavenLocal :photoeditor-ui-default:publishToMavenLocal` and check `~/.m2/repository/com/github/Muzamilabdallah/`.

## Common commands

All commands use the Gradle wrapper at the repo root. (If `java` isn't on PATH, use Android Studio's JBR: `export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"`.)

- Build everything: `./gradlew :photoeditor-core:assembleDebug :photoeditor-ui-default:assembleDebug :app:assembleDebug`
- Install demo on device/emulator: `./gradlew :app:installDebug`
- Unit tests (JVM): `./gradlew :photoeditor-core:testDebugUnitTest` (the real test suite; ui-default and app have none of substance)
- Run a single unit test class: `./gradlew :photoeditor-core:testDebugUnitTest --tests "com.muzamil.photoeditor.geometry.CropGeometryTest"` (append `.methodName` for one test)
- Lint: `./gradlew :photoeditor-core:lintDebug :photoeditor-ui-default:lintDebug :app:lintDebug`
- Clean: `./gradlew clean`

## Architecture

Single-Activity Compose app for editing a picked image by overlaying text, emojis, freehand drawings, and predefined shapes, then exporting the composite to MediaStore.

README.md documents consumer-facing install/wiring for both modes; THEMING.md documents the token system. Don't re-document those here.

### Data flow (Mode B)

`MainActivity`/`DefaultUiSample` (`:app` — owns photo picking and uri persistence) → `PhotoEditor` (public entry, ui-default `ui/PhotoEditor.kt` — theme + default state holder; **no picker**: consumers pass `bitmap` or wire `onPickImage`, nullable = hides pick affordances) → `EditorHostContent` (adapter, `ui/EditorHost.kt`) → `PhotoEditorScreen` (stateless; receives the canvas as a **slot** — core's `EditorCanvas` is `internal`, so the host injects the public `PhotoEditorCanvas(state)`).

Mode A consumers skip all of that and use core's public `PhotoEditorCanvas` (`canvas/PhotoEditorCanvas.kt`) — the editing surface alone, auto-wired to the state holder.

State is owned by `PhotoEditorState` (core, `state/PhotoEditorState.kt`), a plain Compose state holder created via `rememberPhotoEditorState()` — deliberately **not** an AAC ViewModel. It exposes `editorState`/`canUndo`/`canRedo`/`saveSuccess`/`selectedElementBounds`/`isInteracting` as snapshot state; all mutations go through its methods. `isInteracting` is true for as long as a canvas gesture is in flight (freehand stroke, or element drag/rotate/scale) — the canvas reports it, and UIs fade their chrome out while it's set so nothing covers the stroke. Retention across rotation/process death via custom `Saver` (`state/PhotoEditorStateSaver.kt`) — bitmap intentionally excluded (too large); persisting the image *uri* and re-decoding on restore is the consumer's job. Undo history is not retained.

### Shape tool state

`ShapeToolState` (core, `domain/ShapeToolState.kt`): `selectedShape`/`selectedColor`/`strokeWidthPx`/`isFilled` — the pending style for the next shape, held in `EditorState.shapeTool` and mutated via `PhotoEditorState.updateShapeTool(...)`. `EditorTool.Shape` is a plain `data object` marker; the selected shape lives ONLY in `shapeTool.selectedShape` (single source of truth shared by built-in and custom UIs). `state.addShape()` (no args) places the tool's selected shape. `EditorShape` enum: Rect, Circle, Line, Triangle, Star, Heart, Arrow. `EditorColors` (core) is the 8-color shared palette.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Muzamilabdallah/photoEditor](https://github.com/Muzamilabdallah/photoEditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
