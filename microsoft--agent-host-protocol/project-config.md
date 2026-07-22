---
trigger: always_on
description: This directory contains the **Kotlin/JVM** client library for the Agent Host Protocol (AHP), distributed via Maven Central as `com.microsoft.agenthostprotocol:agent-host-protocol`.
---

# Kotlin Client — Agent Guide

## Overview

This directory contains the **Kotlin/JVM** client library for the Agent Host Protocol (AHP), distributed via Maven Central as `com.microsoft.agenthostprotocol:agent-host-protocol`.

The library targets pure Kotlin/JVM (Java 8 bytecode, JDK 17 toolchain) so it works for Android consumers, server-side JVM consumers, and KMP/JVM target consumers without any Android-specific dependencies. Only `kotlinx-serialization-json` is on the classpath at runtime.

## Code Generation

Types in `src/main/kotlin/com/microsoft/agenthostprotocol/generated/` are **auto-generated** from the TypeScript definitions in `types/`. Do not edit these files directly. Generated files are committed to source control so the package is consumable via Maven Central without a code-generation toolchain.

To regenerate after protocol changes:

```bash
npm run generate:kotlin    # runs: tsx scripts/generate.ts --kotlin
```

Generated files: `State`, `Commands`, `Actions`, `Errors`, `Messages`, `Notifications` — all suffixed `.generated.kt`.

CI verifies the committed generated files match the output of `npm run generate:kotlin` and fails on drift.

## Library structure

- `src/main/kotlin/com/microsoft/agenthostprotocol/Ahp.kt` — Hand-maintained entry point. Exposes the configured `kotlinx.serialization.json.Json` instance (`Ahp.json`) that consumers MUST use to encode/decode protocol messages. The custom `KSerializer`s for discriminated unions require a JSON-aware encoder/decoder, so generic `Json` instances may not work.
- `src/main/kotlin/com/microsoft/agenthostprotocol/Reducers.kt` — Hand-written pure reducers (`rootReducer`, `sessionReducer`, `chatReducer`, `terminalReducer`, `changesetReducer`, `annotationsReducer`, `resourceWatchReducer`) ported from `types/channels-*/reducer.ts`, plus a small `Reducer<S, A>` fun-interface and per-channel `object` wrappers. See [Reducers](#reducers) below.
- `src/main/kotlin/com/microsoft/agenthostprotocol/generated/` — Auto-generated wire types.
- `build.gradle.kts` — Gradle build config. Sets `jvmTarget = JVM_1_8` (Android-friendly) with a JDK 17 toolchain. Configures the Vanniktech `maven-publish` plugin for Sonatype Central Portal publishing. Also wires the absolute path of `types/test-cases/reducers/` into the test JVM as the `ahp.reducerFixturesDir` system property so `FixtureDrivenReducerTest` can load fixtures regardless of cwd.
- `gradle.properties` — Source of truth for the artifact's Maven coordinates (`GROUP`, `VERSION_NAME`) and POM metadata.
- `gradle/libs.versions.toml` — Version catalog (Kotlin, kotlinx.serialization, JUnit, Vanniktech plugin).

## Type mapping (TS → Kotlin)

| TypeScript                    | Kotlin                                                            |
| ----------------------------- | ----------------------------------------------------------------- |
| `string`                      | `String`                                                          |
| `number`                      | `Long` (TS contract: 64-bit ints)                                 |
| `number` w/ `@format float`   | `Double`                                                          |
| `boolean`                     | `Boolean`                                                         |
| `unknown` / `object`          | `kotlinx.serialization.json.JsonElement`                          |
| `T \| null`                   | `T?`                                                              |
| `T?` field / `T \| undefined` | `T? = null`                                                       |
| `T[]` / `Array<T>`            | `List<T>`                                                         |
| `Record<string, T>`           | `Map<String, T>`                                                  |
| `Partial<T>`                  | `PartialT` data class with all fields nullable                    |
| `enum E { A = "a" }`          | `@Serializable enum class E { @SerialName("a") A }`               |
| Bitset enum (JSDoc "Bitset")  | `@JvmInline value class` over `Int` w/ companion-object constants |
| Interface struct              | `@Serializable data class`                                        |
| Discriminated union           | sealed interface + custom `KSerializer` (mirrors Swift)           |
| `URI`                         | `typealias URI = String`                                          |
| `StringOrMarkdown`            | sealed interface w/ custom serializer                             |
| Recursive struct              | data class (heap-allocated by default)                            |
| `_meta` field                 | Kotlin `meta` + `@SerialName("_meta")`                            |
| `snake_case` wire field       | camelCase + `@SerialName("snake_case")`                           |

### Why custom union serializers (and not `@JsonClassDiscriminator`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/agent-host-protocol](https://github.com/microsoft/agent-host-protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
