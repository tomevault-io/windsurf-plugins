---
trigger: always_on
description: This repository is the Amazon App Platform: a Kotlin Multiplatform application framework plus example applications and a starter blueprint. The core concepts are documented in [`docs/`](docs/) and implemented across reusable library modules plus a few app entrypoints.
---

# AGENTS.md

## Purpose

This repository is the Amazon App Platform: a Kotlin Multiplatform application framework plus example applications and a starter blueprint. The core concepts are documented in [`docs/`](docs/) and implemented across reusable library modules plus a few app entrypoints.

Start here before changing code:

- `README.md`
- `docs/index.md`
- `docs/setup.md`
- `docs/module-structure.md`
- `docs/di.md`
- `docs/presenter.md`
- `docs/renderer.md`
- `docs/template.md`
- `docs/testing.md`
- `settings.gradle`
- `buildSrc/src/main/kotlin/software/amazon/app/platform/gradle/buildsrc/`

`mkdocs.yml` is the docs site manifest. The Pages workflow builds Wasm artifacts for `:sample:app` and `:recipes:app` and copies them into `docs/web/` before publishing.

## Repo Shape

Important top-level areas:

- `gradle-plugin/`: the published `software.amazon.app.platform` Gradle plugin.
- `buildSrc/`: repo-local convention plugins used by this repository’s own modules. This is where platform targets, emulator config, desktop packaging, and Wasm defaults are defined.
- `docs/`: framework documentation. Treat this as the authoritative product docs.
- `sample/`: the main sample app. This is the best place to study end-to-end usage of scopes, DI, presenters, renderers, templates, fakes, and robots.
- `recipes/`: a second example app plus reusable “recipe” patterns, including the separate `recipesIosApp` SwiftUI/Xcode wrapper.
- `blueprints/starter/`: a standalone starter app template with its own Gradle wrapper, version catalog, and README.

Core framework module families:

- `scope`, `di-common`
- `presenter`, `presenter-molecule`
- `renderer`, `renderer-android-view`, `renderer-compose-multiplatform`
- `robot`, `robot-compose-multiplatform`, `robot-internal`
- `kotlin-inject`, `kotlin-inject-extensions`
- `metro`, `metro-extensions`
- `ksp-common`

Compiler plugin work currently lives in:

- `metro-extensions/contribute/impl-compiler-plugin/`: JVM-only Kotlin compiler plugin module for Metro-backed App Platform DI extensions such as `@ContributesRobot`. `src/main/` contains FIR generation and diagnostics. `src/test/resources/box`, `diagnostics`, and `dump` contain compiler test data. `src/test/java/.../runners/` contains generated JUnit test runners and must be regenerated with `generateTests` after adding or renaming test data files.

## Architecture Rules

The most important repo rule is the module structure documented in `docs/module-structure.md`.

- `:public` modules expose reusable APIs and shared code.
- `:impl` modules contain concrete implementations.
- `:testing` modules hold shared fakes and test helpers.
- `:*-robots` modules hold shared UI robots.
- `:app` modules are the only modules allowed to depend on `:impl` modules.

Do not introduce a dependency from a non-`:app` module to an `:impl` module. The build enforces this via `checkModuleStructureDependencies`.

The framework’s architectural flow is:

1. `Scope` and DI assemble objects for a lifecycle boundary.
2. `MoleculePresenter` implementations produce models.
3. App-specific `Template` presenters wrap the root model tree.
4. `RendererFactory` resolves platform renderers for those models.
5. Thin platform entrypoints bootstrap the root scope and start rendering.

Representative entrypoints:

- Android: `sample/app/src/androidMain/.../AndroidApplication.kt`, `MainActivity.kt`
- iOS: `sample/app/src/iosMain/.../MainViewController.kt`, `sample/iosApp/`
- Desktop: `sample/app/src/desktopMain/.../Main.kt`, `DesktopApp.kt`
- Wasm: `sample/app/src/wasmJsMain/.../Main.kt`

## Toolchain

Local development should match CI as closely as possible. These versions live in `gradle/libs.versions.toml`.

Expected warning: Gradle prints a warning that configuration-on-demand is not supported for Wasm targets. This is noisy but currently normal in this repo.

For Metro compiler-plugin work, prefer source over decompiled artifacts:

- Reference implementation: `https://github.com/square/metro-extensions`
- Metro source: use a local checkout if you have one, otherwise upstream Metro on GitHub
- Avoid relying on `.gradle/caches` or decompiled JARs when the source is available

## Run The Apps

There are three app-style entrypoints to care about:

- `:sample:app`: main sample app inside the root build.
- `:recipes:app`: recipe/demo app inside the root build.
- `blueprints/starter`: standalone starter app; run commands from inside that directory or use its own `./gradlew`.

### Android

Install the debug APK onto a connected device or emulator:

```bash
./gradlew :sample:app:installDebug
./gradlew :recipes:app:installDebug
```

For the standalone starter:

```bash
cd blueprints/starter
./gradlew :app:installDebug
```

`buildSrc/.../BaseAndroidPlugin.kt` configures managed emulator tests with a local device named `emulator` using a Pixel 3 / API 30 `aosp-atd` image.

### iOS

Sample app:

```bash
open sample/iosApp/iosApp.xcodeproj
```

Recipe app:

```bash
open recipes/recipesIosApp/recipesIosApp.xcodeproj
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vRallev/app-platform](https://github.com/vRallev/app-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
