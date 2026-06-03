---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Kotlin External FIR Support (KEFS)** — an IntelliJ IDEA plugin that enables safe loading and management of external Kotlin compiler (FIR) plugins inside the IDE. It intercepts IDE requests for compiler plugins, finds/downloads IDE-compatible versions from Maven repositories, caches them locally, and monitors for runtime exceptions.

Plugin ID: `com.github.mr3zee.kotlinPlugins`

## Build & Development Commands

```bash
# Build the plugin (produces zip in build/distributions/)
./gradlew buildPlugin

# Run tests
./gradlew check

# Run a single test (JUnit 4)
./gradlew test --tests "com.github.mr3zee.kefs.MyPluginTest.testVersionComparison"

# Run the plugin in a sandbox IDE instance (K2 mode enabled)
./gradlew runIde --stacktrace

# Verify plugin compatibility with multiple IDE versions
./gradlew verifyPlugin

# Run for a specific IDE major version (platformVersion and kotlin.lang are auto-resolved from the IDE version map in gradle.properties)
./gradlew runIde -PpluginIdeVersionMajor=253 --stacktrace
```

Requires **JDK 21** (configured via Gradle toolchain). Uses **Gradle 9.2.0** with configuration cache and build cache enabled.

## Architecture

### Core Flow

1. **`KefsProvider`** implements `KotlinBundledFirCompilerPluginProvider` — the entry point called by the Kotlin IDE plugin when it needs a compiler plugin JAR. It matches requested JARs against configured plugin descriptors (via path/artifact-id matching or custom replacement patterns) and delegates to `KefsStorage`.

2. **`KefsStorage`** (project-level service) manages the full plugin lifecycle:
   - In-memory cache (`pluginsCache`) maps plugin names → artifact keys → `ArtifactState`
   - Lifecycle cache prevents redundant lookups within a single IDE analysis pass
   - Orchestrates JAR resolution: checks disk cache first, then triggers `KefsJarLocator`
   - Runs file watchers (via `WatchService`) on both cached and original (local repo) JAR directories for hot-reload
   - Handles cache invalidation with a debounce mechanism (workaround for KTIJ-37664)
   - Manages periodic auto-update via configurable interval

3. **`KefsJarLocator`** handles artifact resolution:
   - Fetches Maven `maven-metadata.xml` from remote URLs or local paths
   - Applies version matching (Exact/Same Major/Latest) across artifact bundles
   - Downloads JARs with checksum verification (MD5)
   - Supports `for-ide` classifier fallback
   - Local repository JARs are copied and symlinked for change tracking

4. **`KefsSettings`** (persistent project-level service, stored in `.idea/kotlin-plugins.xml`) holds repositories and plugin descriptors. Default state is loaded from `defaults.xml` via `KefsDefaultStateLoader`.

5. **`KefsExceptionAnalyzerService`** monitors IDE exceptions and matches stack traces against loaded plugin classes (analyzed by `KefsJarAnalyzer`) to identify which plugin caused a crash.

### Requested, Resolved, and Kotlin versions
'Version' word of an artifact can relate to different things.
All compiler plugins have a Kotlin and a Library version. By default, an artifact looks like this:
```
<group-id>:<artifact-id>-<kotlin-version>-<lib-version>
```
Example:
```
org.jetbrains.kotlinx:compiler-plugin-k2-2.2.0-ij251-78-0.10.2.jar
```
Here `2.2.0-ij251-78` is the Kotlin version and `0.10.2` is the Library version.

Library version can be requested or resolved:
- Requested – the version that comes from `KotlinBundledFirCompilerPluginProvider`
- Resolved – the version found by `KefsJarLocator` using the version matching strategy.

Kotlin version can be project version or IDE version:
- Project version – the one comes from `KotlinBundledFirCompilerPluginProvider`, same place as a Requested Library version.
- IDE Version – the one from `KotlinVersionService` and for which we look for artifacts.

### Key Domain Types (in `versions.kt`)

- `RequestedVersion` / `ResolvedVersion` — inline value classes distinguishing user-project library version from the resolved IDE-compatible library version
- `JarId` — identifies a loaded JAR (plugin name + maven ID + requested version); equality ignores resolved version
- `VersionedKotlinPluginDescriptor` — a plugin descriptor bound to a specific requested version
- `MatchFilter` — version matching strategy + requested version, used by `getMatching()`

### IDE-Version-Specific Code

The plugin supports multiple IntelliJ platform versions. `VersionSpecificApi` defines the interface; each `src/{251,252,253,261}/kotlin/.../VersionSpecificApiImpl.kt` provides the implementation. The active source set is selected at build time based on `pluginSinceBuild` in `gradle.properties`.

### Multi-Version Release

`pluginIdeVersionMajor` in `gradle.properties` controls version-specific builds. When set (e.g., `251`), it autopopulates `sinceBuild`, `untilBuild`, version suffix, `platformVersion`, and Kotlin compiler version from the `ide.<major>.*` map in `gradle.properties`. The CI release dispatcher (`release-dispatcher.yml`) reads the IDE versions from this map and triggers parallel release jobs.

### Adding a New IDE Major Version


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mr3zee/Kotlin-External-FIR-Support](https://github.com/Mr3zee/Kotlin-External-FIR-Support) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
