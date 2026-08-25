---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**kuri** — a URI/URL parsing and manipulation library for Java and Kotlin.

## Current state

kuri is an implemented **Kotlin Multiplatform** library (no longer greenfield), built as three
Gradle modules (declared in `settings.gradle.kts`) that all apply the full quality-gate stack
(ktlint, detekt, explicit-API strict mode, binary-compatibility-validator, Kover):

- **`:kuri`** — the core engine and the only module published to every target (see
  **Targets** below). Source lives under `src/commonMain` — a pure-Kotlin parsing engine with
  `host`/`idna`/`percent`/`query`/`scheme`/`serialize`/`parser` subpackages — with tests in
  `src/commonTest` and JVM-interop tests in `src/jvmTest`.
- **`:kuri-bind`** — a JVM-only reflective annotation binder (`@Url`/`@Uri` roots → `Url`/`Uri`
  builders) that depends on `:kuri`.
- **`:kuri-serde-kotlinx`** — a `kotlinx.serialization` bridge (`Url`/`Uri` serializers, a
  query-parameters format) that depends on `:kuri`; multiplatform across the same
  jvm/js/wasmJs/native targets as `:kuri`, but does not publish an Android variant.

`:kuri-bind` and `:kuri-serde-kotlinx` each pin their own Java-8-bytecode toolchain override
(`jvmTarget = JVM_1_8` / `-Xjdk-release=1.8` / `sourceCompatibility`/`targetCompatibility` — see
**Toolchain** below) that must be kept in sync with `:kuri`'s. A Gradle (Kotlin DSL) build with
the `gradlew` wrapper is committed and drives all three modules.

- **Build & test:** `./gradlew build` runs the whole gate across every module and target. For
  a fast inner loop, scope to a module and the JVM target, e.g. `./gradlew :kuri:jvmTest
  :kuri:ktlintCheck :kuri:detekt :kuri:apiCheck` (swap the `:kuri` prefix for `:kuri-bind` or
  `:kuri-serde-kotlinx` to run the same gate on those modules). The JS/Wasm suites run on Node
  via `:kuri:jsNodeTest` / `:kuri:wasmJsNodeTest` and their `:kuri-serde-kotlinx` equivalents;
  `:kuri-bind` has no JS/Wasm/native tasks since it only targets the JVM.
- **Targets (`:kuri`):** `jvm` (Java 8 bytecode), `android` (namespace `org.dexpace.kuri`,
  `compileSdk = 35`, `minSdk = 21`, published as its own Maven variant), `js` and `wasmJs`
  (browser + Node), and native — `linuxX64`/`linuxArm64`, `macosArm64`, `mingwX64`,
  `iosX64`/`iosArm64`/`iosSimulatorArm64`, `watchosArm64`/`watchosSimulatorArm64`, and
  `tvosArm64`/`tvosSimulatorArm64`. The Android Gradle KMP library plugin doesn't auto-wire
  `apiCheck`/`apiDump` for its single-variant `android` target (upstream gap KT-71172), so
  `kuri/build.gradle.kts` hand-registers `androidApiBuild`/`androidApiCheck`/`androidApiDump`
  tasks against the `kuri/api/android/kuri.api` snapshot and hooks them into the normal
  `apiCheck`/`apiDump` aggregates — a public-API change still means running `apiDump` and
  committing the regenerated snapshot like any other target, but the wiring behind it is
  manual rather than BCV's usual auto-configuration.

The docs can still lag the code — when a detail matters, confirm it against the actual
`kuri/build.gradle.kts` and `gradle/libs.versions.toml` rather than assuming.

## Toolchain

- **Build JDK:** Corretto 21 (the JDK the build runs on, pinned via
  `gradle/gradle-daemon-jvm.properties` and used by every CI workflow); the JVM target pins
  `jvmToolchain(21)` for compilation.
- **Kotlin:** 2.4.0, with language/API version pinned to **2.0** (Kotlin 2.4 dropped 1.9
  support, so 2.0 is the floor the library compiles against).
- **JVM bytecode target:** 1.8 — enforced via `jvmTarget = JVM_1_8` and `-Xjdk-release=1.8`
  so newer-JDK stdlib symbols cannot leak into a library that must run on Java 8. Keep this
  in mind for a published library: avoid APIs unavailable on the Java 8 runtime in shipped
  code (`InputStream.transferTo` (9+), `Thread.threadId()` (19+), records, sealed `permits`,
  etc.).
- **Cross-compile toolchain discipline:** a module that genuinely needs a newer JDK must
  override **all three** of `jvmToolchain(N)`, the `java { sourceCompatibility /
  targetCompatibility }` block, and `compilerOptions { jvmTarget.set(JvmTarget.JVM_N) }`.
  Overriding only the toolchain emits Java-8-format bytecode that references newer stdlib
  symbols (`NoSuchMethodError` on JDK 8).

## Target build setup & conventions

kuri sits in the **Dexpace** family alongside `dexpace/java-sdk`, which it mirrors. These
conventions are wired into the build and are authoritative:

- **Build tool:** Gradle (Kotlin DSL). `gradle/libs.versions.toml` is the single source of
  truth for dependency and plugin versions. Group `org.dexpace`.
- **Quality gates wired into `check` (all break the build):** `ktlint` + `detekt`,
  `allWarningsAsErrors`, **explicit-API strict mode**, **binary-compatibility-validator**
  (`apiCheck`/`apiDump`), and a Kover line-coverage floor. Run `./gradlew build` to execute
  the full gate; `./gradlew apiDump` only after an **intentional** public-API change, and
  commit the regenerated `api/*.api` snapshots alongside it. Regenerate the native
  `kuri/api/kuri.klib.api` dump on **macOS**: running `apiDump` on a Linux/Windows host

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dexpace/kuri](https://github.com/dexpace/kuri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
