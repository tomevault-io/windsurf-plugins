---
trigger: always_on
description: TestBalloon is a DSL-based, Kotlin-first test framework. It supports all Kotlin target platforms (JVM, JS, WebAssembly, Native, Android). This document guides AI agents working in this repository.
---

# Agent Guidelines for TestBalloon

TestBalloon is a DSL-based, Kotlin-first test framework. It supports all Kotlin target platforms (JVM, JS, WebAssembly, Native, Android). This document guides AI agents working in this repository.

## Important

This project requires context beyond the contents of this repository, as indicated in `documentation/Kotlin_IDE_and_Build_Tool_Interoperability.adoc`.

Everything – including issues, pull request messages, comments, code – should be written with brevity and clarity.

## Repository Structure

| Path                            | Description                                                                |
|---------------------------------|----------------------------------------------------------------------------|
| `testBalloon-framework-core/`   | Core framework module (Kotlin Multiplatform)                               |
| `testBalloon-framework-shared/` | Shared internal framework utilities                                        |
| `testBalloon-compiler-plugin/`  | Kotlin compiler plugin that discovers top-level test suites                |
| `testBalloon-gradle-plugin/`    | Gradle plugin for test discovery and configuration                         |
| `testBalloon-integration-*/`    | Optional integrations (Kotest assertions, Robolectric, blocking detection) |
| `build-logic/`                  | Convention plugins for the build (compiled separately)                     |
| `build-settings/`               | Settings-level Gradle plugins                                              |
| `integration-test/`             | End-to-end integration tests using project templates                       |
| `examples/`                     | Example projects demonstrating TestBalloon usage                           |
| `comparisons/`                  | Side-by-side comparisons with other test frameworks                        |
| `experiments/`                  | Experimental/scratch module; not part of the public API                    |
| `documentation/`                | Documentation website (Material for MkDocs) and Dokka plugins              |

## Build System

This project uses **Gradle with Kotlin DSL** (`build.gradle.kts`, `settings.gradle.kts`). The build uses three composite builds:

- **`build-settings/`** – Settings plugin (`buildSettings`), applied in every `settings.gradle.kts`.
- **`build-logic/`** – Convention plugins (e.g. `buildLogic.common`, `buildLogic.kotlin-multiplatform`).
- **Root project** – The main multi-project build.

Versions are managed via `gradle/libs.versions.toml` (version catalog). `versions.properties` must remain unchanged.

## Essential Commands

Always use the Gradle wrapper (`./gradlew`). Do **not** invoke `gradle` directly.

```bash
# Run all component tests (all Kotlin targets)
./gradlew componentTestsAllTargets

# Run component tests (JVM only — faster)
./gradlew componentTestsJvmOnly

# Run integration tests
./gradlew integrationTests

# Run a single subproject's tests (e.g. framework core)
./gradlew :testBalloon-framework-core:allTests

# Run compiler plugin tests
./gradlew :testBalloon-compiler-plugin:test

# Run Gradle plugin tests
./gradlew :testBalloon-gradle-plugin:test

# Check ABI compatibility
./gradlew checkKotlinAbi

# Lint (kotlinter)
./gradlew lintKotlin

# Auto-fix lint issues
./gradlew formatKotlin

# Upgrade JS/Wasm package lock files (required before tests if lock files changed)
./gradlew kotlinUpgradePackageLock kotlinWasmUpgradePackageLock
```

## Coding Conventions

- **Kotlin only.** All production and test code is written in Kotlin.
- Follow the **official Kotlin code style** (`kotlin.code.style=official` in `gradle.properties`).
- Code style is enforced by **kotlinter**. Run `./gradlew formatKotlin` before committing.
- Commit sign-off is required (`web_commit_signoff_required: true`).
- Source sets follow the standard KMP layout: `commonMain`, `jvmMain`, `jsMain`, `nativeMain`, etc.

## Versioning

- Releases are versioned as `<testBalloon-version>-K<kotlin-version>`, e.g. `0.8.4-K2.3.20`.
- Each TestBalloon release ships **multiple variants** supporting different Kotlin versions (see `CHANGELOG.md`).
- The current development version is set in `gradle.properties` (`version=...`). Do not change this unless specifically requested.

## Testing Notes

- **Component tests** cover the framework, compiler plugin, Gradle plugin, and integrations.
- **Integration tests** (`:integration-test`) spin up separate Gradle project templates; they take longer to run.
- On **Windows**, integration tests skip non-ASCII pattern tests (handled by CI automatically).
- CI runs on **ubuntu-latest** (primary), then **macos-latest** and **windows-latest** (secondary, after primary passes).
- When adding tests, follow the style used by existing tests in the relevant subproject.

## PR instructions

- Before preparing a PR, create an issue which clearly and concisely describes the problem to solve and the suggested approach.
- Before preparing a larger PR or a PR doing substantial changes, request feedback on its issue(s) from persons maintaining the repository.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [infix-de/testBalloon](https://github.com/infix-de/testBalloon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
