---
trigger: always_on
description: This document captures how the app is structured today so you can change it confidently. Keep it close when you explore the code base.
---

# CODEX – ShaderEditor

This document captures how the app is structured today so you can change it confidently. Keep it close when you explore the code base.

---

## 1. What the app does
- ShaderEditor is an Android GLSL playground/live wallpaper. Users edit fragment shaders, preview them live, import textures/uniforms, and set any shader as a wallpaper (`README.md`).
- The whole product lives in a single Android application module under `app/`. Source is Java (language level 17) with Gradle Kotlin build scripts.
- The UI is a single-activity, multi-fragment setup with custom widgets for the editor, shader preview, uniform pickers, and texture tooling.
- Sensors, camera, microphone, battery status, wallpaper offsets, notification counts, etc., are exposed to shaders as uniforms via `ShaderRenderer`.

---

## 2. Build, run, and tooling
- Requirements: Android Studio Ladybug+, Android SDK 21–36, Java 17, and the Android NDK is not required. Gradle wrapper + Kotlin DSL (`build.gradle.kts`, `app/build.gradle.kts:1`).
- Build variants: `debug` and `release`; release is minified/resource-shrunk and expects signing env vars (`ANDROID_KEY_*`).
- View binding + `BuildConfig` generation are enabled. Compose is not used.
- Helpful commands:
  - `./gradlew assembleDebug` (or `make debug`) – build APK.
  - `./gradlew assembleRelease` and `./gradlew bundleRelease` – ship builds.
  - `./gradlew lintDebug`, `make lint`, `make infer`, `make avocado` – lint/static analysis/drawable checks.
  - `make install`, `make start`, `make uninstall` – adb helpers.
- Version catalog (`gradle/libs.versions.toml:1`) centralizes dependency versions (AppCompat 1.7, Material 1.12, Preference 1.2, CameraX 1.4, AGP 8.12).
- Fastlane metadata lives in `fastlane/` for Play/F-Droid; SVG assets are under `svg/`.

---

## 3. Repository layout (high signal paths)

| Path | Notes |
| --- | --- |
| `app/src/main/AndroidManifest.xml:1` | Permissions (camera, mic, legacy storage), wallpaper + notification services, exported activities. |
| `app/src/main/java/de/markusfisch/android/shadereditor/` | All runtime code. Key packages: `activity`, `fragment`, `widget`, `opengl`, `database`, `hardware`, `io`, `view`, `adapter`. |
| `app/src/main/res/` | Layouts, drawables, fonts, `res/raw/*.glsl` sample shaders, `res/xml/preferences.xml`. |
| `res/xml/backup_rules.xml`, `res/xml/extraction_rules.xml` | Backup configuration referenced by the manifest. |
| `fastlane/metadata/...` | Store descriptions/screenshots. Keep in sync for releases. |
| `CONTRIBUTING.md`, `FAQ.md`, `CHANGELOG.md` | Ground rules, docs referenced in-app. |

---

## 4. Runtime architecture

### 4.1 Application start
- `ShaderEditorApp` (`app/src/main/java/de/markusfisch/android/shadereditor/app/ShaderEditorApp.java:1`) is the `Application`. It initializes the singleton `Preferences`, prewarms `Database`, keeps a process-wide `UndoRedo.EditHistory`, turns on StrictMode in debug builds, and registers `BatteryLevelReceiver` (API ≥ 24) to drive low-power behavior.
- Splash flow: `SplashActivity` immediately launches `MainActivity` so the app initializes before the UI is drawn (`activity/SplashActivity.java:1`).

### 4.2 Main screen plumbing
- `MainActivity` (`activity/MainActivity.java:1`) hosts everything. It instantiates:
  - `EditorFragment` (code editing).
  - `ShaderViewManager` (GLSurfaceView preview + quality spinner).
  - `ShaderManager` (load/save shaders, handle `ACTION_SEND`/`ACTION_VIEW` intents, duplicate/delete, persist thumbnails).
  - `ShaderListManager` (ListView + background loader for saved shaders).
  - `UIManager` (toolbar buttons, drawer, toggling editor visibility, extra keys).
  - `MainMenuManager` (popup menu with editor actions, navigation to Add Uniform, samples, settings).
  - `NavigationManager` (previews, FAQ link, share intent).
  - `ExtraKeysManager` (soft keyboard helpers + completions strip).
- State flow: the editor notifies `ShaderManager` when text changes; `ShaderManager` updates `ShaderViewManager` and `ShaderListManager`; UI state (title/subtitle, show-errors pill) is adjusted via `UIManager`.

### 4.3 Editor stack
- `EditorFragment` (`fragment/EditorFragment.java:1`) wires the custom `ShaderEditor` widget to undo/redo, completion listeners, and preference-driven appearance (font, ligatures, line numbers, update delay).
- `ShaderEditor` (`widget/ShaderEditor.java:1`) extends `LineNumberEditText`. Responsibilities: syntax highlighting (`highlighter/*`), debounce compile requests, auto-insert braces, convert ShaderToy snippets, completion entry, lint error highlighting. Highlights/errors are recalculated on a worker thread (`TokenListUpdater`).
- `UndoRedo` (`view/UndoRedo.java:1`) tracks a shared, process-wide edit history so the editor can preserve undo stacks between fragment recreations (history object is kept in `ShaderEditorApp.editHistory`).
- `ExtraKeysManager` (`activity/managers/ExtraKeysManager.java:1`) shows tab/brace buttons and completion chips, auto-hiding when the IME is closed if preferences demand.
- Compile errors from the GL renderer are visualized by `ErrorListModal` and `ErrorAdapter` (`widget/ErrorListModal.java:1`, `adapter/ErrorAdapter.java:1`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markusfisch/ShaderEditor](https://github.com/markusfisch/ShaderEditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
