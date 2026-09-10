---
trigger: always_on
description: This file provides guidance to coding agents (Claude Code, Codex, and other AI assistants) working in this repository.
---

# AGENTS.md

This file provides guidance to coding agents (Claude Code, Codex, and other AI assistants) working in this repository.

## Project Overview

Kotlin Multiplatform SDK for the DeepSeek REST API.

- Published coordinates: `org.oremif:deepseek-kotlin`
- Module version in source: `0.4.0`
- Root Gradle build includes only `:deepseek-kotlin`
- `example/` is a separate standalone Gradle project with its own wrapper and `settings.gradle.kts`

Supported targets: JVM, Android, Apple (`iosX64`, `iosArm64`, `iosSimulatorArm64`, `macosArm64`), Linux (`linuxX64`, `linuxArm64`), MinGW (`mingwX64`), WebAssembly (`wasmJs`).

## Project Structure

- `deepseek-kotlin/` — main published library module (Kotlin sources under `src/commonMain/kotlin/org/oremif/deepseek/`)
  - `api/` — endpoint free functions: `chat.kt`, `chatStream.kt`, `fimCompletion.kt`, `fimCompletionStream.kt`, `userBalance.kt`, `models.kt`
  - `client/` — `DeepSeekClient`, `DeepSeekClientStream`, `DeepSeekClientConfig`, `LoggingConfig`
  - `models/` — request/response data classes, enums, message types
  - `errors/` — `DeepSeekError`, `DeepSeekHeaders`
  - `utils/` — `retry.kt` (retry policy), `validate.kt`
  - `src/commonTest/kotlin/` — shared tests; mock infrastructure in `org/oremif/deepseek/testing/MockClients.kt`
- `deepseek-kotlin/api/` — locked ABI snapshots for `checkKotlinAbi` (JVM, Android, klib)
- `example/` — standalone JVM sample consuming the published artifact
- `gradle/libs.versions.toml` — centralized version catalog
- `.github/workflows/` — CI (`ci.yml`) and Dokka-to-GitHub-Pages (`docs.yml`)

## Core Architecture

1. **Client layer** — `DeepSeekClientBase` holds a shared `HttpClient` + `DeepSeekClientConfig`. `DeepSeekClient` handles unary calls; `DeepSeekClientStream` layers SSE on top for streaming (returns `Flow`). Both use a builder that assembles the Ktor client lazily in `build()`, so `jsonConfig` / `httpClient { }` blocks compose regardless of ordering. Clients are designed to be long-lived — `close()` is rarely needed; use `closeAndJoin()` from a coroutine for graceful shutdown.
2. **Default Ktor plugins** — `Auth` (bearer), `ContentNegotiation` (JSON, `SnakeCase` naming, `ignoreUnknownKeys=true`), `defaultRequest` pinning base URL `https://api.deepseek.com`, `HttpRequestRetry` (3 retries, honors `Retry-After`, retries on timeouts and statuses classified by `utils/retry.kt`), `HttpTimeout` (request 60s, connect 10s, socket 300s), optional `Logging`.
3. **Logging is opt-in.** Nothing is logged unless `logging { }` is called. `Authorization` is always redacted; callers add further `sanitizeHeader { }` predicates as needed.
4. **Per-platform HTTP engines** — JVM/Android: OkHttp; Apple (iOS + macOS): Darwin; Linux/MinGW: CIO; wasmJs: JS. Engine deps are declared in `deepseek-kotlin/build.gradle.kts` source sets.
5. **Endpoint files are free functions** on `DeepSeekClient` / `DeepSeekClientStream` (not methods) — look them up in `api/*.kt` rather than on the client class.
6. **`explicitApi()` is enabled** — every public declaration must have an explicit visibility and return type.

## Common Commands

### Build

```bash
./gradlew build                      # all targets
./gradlew :deepseek-kotlin:build     # library only
./gradlew -p example build           # standalone example project
```

### Test

```bash
./gradlew :deepseek-kotlin:jvmTest              # JVM (fastest; what CI runs)
./gradlew :deepseek-kotlin:test                 # all configured targets
./gradlew :deepseek-kotlin:wasmJsNodeTest       # WasmJS via Node
./gradlew :deepseek-kotlin:wasmJsBrowserTest    # WasmJS in headless browser

# Run a single test class / method (Gradle test filter):
./gradlew :deepseek-kotlin:jvmTest --tests "org.oremif.deepseek.api.ChatCompletionApiTests"
./gradlew :deepseek-kotlin:jvmTest --tests "*ChatCompletionApiTests.someMethodName"
```

Tests use `kotlin.test` + Kotest assertions (`io.kotest.matchers.*`).

### Formatting (ktfmt)

```bash
./gradlew ktfmtFormat ktfmtFormatScripts   # reformat sources + build scripts
./gradlew ktfmtCheck ktfmtCheckScripts     # verify (what the `lint` CI job runs)
./gradlew -p example ktfmtFormat ktfmtFormatScripts   # standalone example project
```

Run the format tasks after editing Kotlin code; CI fails otherwise. ktfmt is non-configurable by design, so
style debates are settled by the tool.

### Binary compatibility (ABI) validation

ABI validation is built into the Kotlin Gradle plugin (`kotlin { abiValidation() }`, experimental DSL), so no separate `binary-compatibility-validator` plugin is applied. It locks the public API for the JVM, Android and klib targets; intentional API changes must be accompanied by regenerated dumps under `deepseek-kotlin/api/` (`jvm/`, `android/` and the shared `.klib.api`).

```bash
./gradlew :deepseek-kotlin:checkKotlinAbi     # fails if public API drifts from the locked dumps
./gradlew :deepseek-kotlin:updateKotlinAbi    # regenerate dumps after an intentional API change
```

`checkKotlinAbi` is wired into the `check` lifecycle task. The old `checkLegacyAbi` / `updateLegacyAbi` names still exist as deprecated aliases — don't use them.

### Documentation

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Oremif/deepseek-kotlin](https://github.com/Oremif/deepseek-kotlin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
