---
trigger: always_on
description: - Keep a `CURRENT_CHAT_CONTEXT` file with the current chat flow so work can be recovered after session failures.
---

# AGENTS Guide for `gdx-teavm`

## Mandatory Notes
- Keep a `CURRENT_CHAT_CONTEXT` file with the current chat flow so work can be recovered after session failures.
- `CURRENT_CHAT_CONTEXT` is for last-chat context only; do not keep long-term history there.
- At session start, always check whether `CURRENT_CHAT_CONTEXT` exists; if it does, load it before proceeding.
- Always validate that a class, member, method, task, or Gradle property already exists before using it. Never guess API names or signatures.
- Never write or modify any file without explicit user permission. Planning and review steps are allowed without writing.
- After making changes, run a simple Gradle build to confirm code still compiles. Prefer the smallest affected module/task, or `./gradlew build` when unclear.

## Common Workflows
- Inspect project graph:
  - `./gradlew projects`
- Build everything:
  - `./gradlew build`
- Compile the Gradle plugin:
  - `./gradlew :gdx-teavm-plugin:compileKotlin`
- Compile the shared/web backends:
  - `./gradlew :backends:backend-shared:compileJava :backends:backend-web:compileJava`
- Compile native backends:
  - `./gradlew :backends:backend-glfw:compileJava :backends:backend-ios:compileJava`

## Example Tasks
- Manual builder web example:
  - `./gradlew :examples:basic:platforms:web:builder:basic_web_run`
- Manual builder desktop C example:
  - `./gradlew :examples:basic:platforms:desktop:teavm-c:builder:basic_desktop_c_generate`
  - `./gradlew :examples:basic:platforms:desktop:teavm-c:builder:basic_desktop_c_debug_build`
  - `./gradlew :examples:basic:platforms:desktop:teavm-c:builder:basic_desktop_c_debug_run`
- Gradle plugin basic example:
  - `./gradlew :examples:basic:platforms:web:plugin:gdx_teavm_web_js_run`
  - `./gradlew :examples:basic:platforms:web:plugin:gdx_teavm_web_wasm_run`
  - `./gradlew :examples:basic:platforms:desktop:teavm-c:plugin:gdx_teavm_glfw_generate`
  - `./gradlew :examples:basic:platforms:desktop:teavm-c:plugin:gdx_teavm_glfw_build`
  - `./gradlew :examples:basic:platforms:desktop:teavm-c:plugin:gdx_teavm_glfw_run`
- Gradle plugin FreeType web example:
  - `./gradlew :examples:freetype:platforms:web:plugin:gdx_teavm_web_js_run`
  - `./gradlew :examples:freetype:platforms:web:plugin:gdx_teavm_web_wasm_run`
- Gradle plugin gdx-controllers web example:
  - `./gradlew :examples:controllers:platforms:web:plugin:gdx_teavm_web_js_run`
  - `./gradlew :examples:controllers:platforms:web:plugin:gdx_teavm_web_wasm_run`
- Publishing entry points are root tasks provided by `com.github.xpenatan.easy-publishing` and shown under the `easy-publishing` task group:
  - `prepareSnapshot`
  - `prepareRelease`
  - `publishSnapshot`
  - `publishRelease`

## Project Shape
- This is a multi-module Gradle build. Active modules live under `backends/`, `extensions/`, `tools/gdx-teavm-plugin`, and `examples/`.
- Each example keeps portable code in `examples/<name>/core`, optional shared runtime assets in `examples/<name>/assets`, and runnable launchers under `examples/<name>/platforms`.
- Desktop implementations are grouped under `platforms/desktop`; TeaVM C and web use `builder` and `plugin` leaves only when both build styles exist.
- Intermediate example directories are organizational parents. Runnable platform leaves depend directly on their example's `core`, not on sibling platform projects.
- `settings.gradle.kts` includes the local Gradle plugin build with `includeBuild("tools/gdx-teavm-plugin")`; the included build appears as project path `:gdx-teavm-plugin` in Gradle output.
- Version numbers and build switches are centralized in `buildSrc/src/main/kotlin/LibExt.kt`.
- Root `build.gradle.kts` applies shared Java 11 settings and Maven repositories to subprojects.
- `gradle.properties` can enable composite builds for local libGDX or TeaVM source:
  - `includeLibgdxSource`
  - `includeTeaVMSource`
  - `gdxSourcePath`
  - `teavmPath`

## Core Compiler Pipeline
- The manual builder API lives in `backends/backend-shared`.
- `TeaBuilder` is the fluent public entry point. It stores configuration in `TeaBuilderData`.
- `TeaBuilder.build(File output)` delegates to a concrete `TeaBackend`.
- `TeaBackend.compile(...)` performs shared setup:
  - classpath collection
  - TeaVM tool configuration
  - reflection metadata setup
  - asset planning and copying
  - backend-specific build hooks
- Put behavior in `TeaBackend` only when it is shared by all targets. Keep target-specific behavior in:
  - `WebBackend`
  - `TeaGLFWBackend`

## Backends
- `backends/backend-web`
  - Runtime classes: `WebApplication`, `WebApplicationConfiguration`, WebGL/audio/filesystem implementations.
  - Builder backend: `WebBackend`.
  - Targets JavaScript or Wasm based on `WebBackend.setWebAssembly(boolean)`.
  - Generates `index.html`, `WEB-INF/web.xml`, web assets, preload manifest, scripts, and optional Jetty serving.
- `backends/backend-glfw`
  - Runtime classes: `GLFWApplication`, `GLFWApplicationConfiguration`.
  - Builder backend: `TeaGLFWBackend`.
  - Targets TeaVM C output and writes a native CMake project with debug/release build scripts.
  - Output is typically under `build/dist/glfw/c` in plugin mode or `build/dist/c` in builder mode.
## Gradle Plugin

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xpenatan/gdx-teavm](https://github.com/xpenatan/gdx-teavm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
