---
trigger: always_on
description: Booster is an open-source Android application quality optimization framework from DiDi, implementing extensible bytecode transformation and task processing through Gradle plugins + SPI mechanism.
---

# Booster Project Guide

## Project Overview

Booster is an open-source Android application quality optimization framework from DiDi, implementing extensible bytecode transformation and task processing through Gradle plugins + SPI mechanism.

**Core Features**: Performance detection, system bug fixes, multi-threading optimization, app size reduction (resource deduplication, R class inlining, etc.)

**Results**: 15%~25% stability improvement, 1MB~10MB package size reduction

## Tech Stack

- **Language**: Kotlin (primary) + Java
- **Build**: Gradle 8.5 + Kotlin DSL
- **Bytecode**: ASM + Javassist 3.30.2-GA
- **Compatibility**: AGP 8.0 ~ 9.0 (v8_0, v8_1, v8_2, v8_3, v8_4, v8_5, v8_6, v8_7, v8_8, v8_9, v8_10, v8_12, v9_0)
- **Publishing**: Maven Central (com.didiglobal.booster)

## Module Architecture

### Core Framework
- `booster-gradle-plugin` - Gradle plugin entry point
- `booster-api` - Core API and utilities
- `booster-annotations` - Annotation definitions

### SPI Layer (Extension Interfaces)
- `booster-transform-spi` - Bytecode transformation interfaces (`Transformer`, `TransformContext`, `KlassPool`)
- `booster-task-spi` - Task processor interface (`VariantProcessor`)

### Bytecode Transformation Tools
- `booster-transform-asm` - ASM implementation
- `booster-transform-javassist` - Javassist implementation
- `booster-transform-util` - Utility library

### AGP Compatibility Layer
- `booster-android-gradle-api` - API abstraction
- `booster-android-gradle-compat` - Compatibility support
- `booster-android-gradle-v8_*` - AGP version adapters (v8_0 ~ v8_12)
- `booster-android-gradle-v9_0` - AGP 9.0 adapter

### Transform Implementations
- `booster-transform-thread` - Multi-threading optimization
- `booster-transform-toast` - Toast crash fix
- `booster-transform-r-inline` - R class constant inlining
- `booster-transform-res-check` - Resource checking
- `booster-transform-activity-thread` - ActivityThread optimization
- `booster-transform-finalizer-watchdog-daemon` - Finalizer optimization
- `booster-transform-logcat` - Logcat optimization
- `booster-transform-media-player` - MediaPlayer optimization

### Runtime Instrumentation
- `booster-android-instrument-*` - Runtime hook implementations for each feature

### Auxiliary Tools
- `booster-cha` / `booster-cha-asm` - Class Hierarchy Analysis
- `booster-graph-*` - Task graph visualization
- `booster-task-list-*` - Task list tools
- `booster-aapt2` - AAPT2 integration

## Key Entry Files

| File | Path | Purpose |
|------|------|---------|
| `BoosterPlugin.kt` | `booster-gradle-plugin/.../gradle/` | Plugin main class, implements `Plugin<Project>` |
| `BoosterTransformTask.kt` | same | Bytecode transformation task |
| `Transformer.kt` | `booster-transform-spi/.../transform/` | Transformer interface definition |
| `TransformContext.kt` | same | Transform context |
| `VariantProcessor.kt` | `booster-task-spi/.../task/spi/` | Variant processor interface |
| `AsmTransformer.kt` | `booster-transform-asm/.../asm/` | ASM transformer implementation |
| `ClassTransformer.kt` | same | Class transformer abstract class |

## Common Commands

```bash
# Build project
./gradlew build

# Publish to local Maven
./gradlew publishToMavenLocal

# Run tests
./gradlew test

# Run integration tests
./gradlew integrationTest

# Compile integration tests (verify compatibility)
./gradlew compileIntegrationTestKotlin

# View all tasks
./gradlew tasks
```

## AGP Version Compatibility Development

### Kotlin 2.0 Metadata Compatibility

AGP 8.9+ is compiled with Kotlin 2.0, producing Kotlin metadata incompatible with Kotlin 1.x. Solution:

1. **Gradle Version**: Use Gradle 8.5+ (includes Kotlin 1.9.20 which better handles Kotlin 2.0 metadata)
2. **Compiler Args**: Add to v8_9+ module's `build.gradle`:
   ```groovy
   tasks.withType(org.jetbrains.kotlin.gradle.tasks.KotlinCompile).configureEach {
       kotlinOptions {
           freeCompilerArgs += ['-Xskip-metadata-version-check']
       }
   }
   ```

### AGP Version to Gradle Version Mapping

| AGP Version | Minimum Gradle Version |
|-------------|------------------------|
| 8.0 - 8.8   | 8.0+                   |
| 8.9 - 8.11  | 8.11.1+                |
| 8.12        | 8.13+                  |
| 9.0         | 9.1+ (not yet released)|

### Adding New AGP Version Support

1. Create module `booster-android-gradle-v{major}_{minor}`
2. Copy existing module structure (e.g., v8_2)
3. Update AGP dependency version and SDK tools version
4. Implement `AGPInterface`, handling API changes
5. Create `src/integrationTest/` directory with:
   - `kotlin/.../V{major}{minor}IntegrationTest.kt` - Test class
   - `resources/` - Test resources (app.gradle, lib.gradle, buildSrc/, src/)
6. Create `src/e2eTest/` directory with standalone Android test project
7. Update `GradleExecutor` version in integration test to match AGP requirements

### Integration Test Structure

Each AGP adapter module contains:
- `src/integrationTest/` - Gradle TestKit integration tests
- `src/e2eTest/` - Standalone Android project for end-to-end testing

### Running Integration Tests

Integration tests require:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [didi/booster](https://github.com/didi/booster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
