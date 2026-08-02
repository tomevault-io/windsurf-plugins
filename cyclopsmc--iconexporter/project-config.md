---
trigger: always_on
description: This document provides essential information for developers and AI agents working on this Minecraft mod's codebase.
---

# AGENTS.md - Developer and AI Agent Guide

This document provides essential information for developers and AI agents working on this Minecraft mod's codebase.

## Project Overview

This is a Minecraft mod. See README.md for a description of what this mod does. The project is written in Java and uses Gradle as its build system.

## Multi-Loader Architecture

**This repo only uses a multi-loader architecture if you can find a `loader-common` directory in the root of this repo!**

In Minecraft 1.21 and above, this repository uses a **multi-loader setup**, meaning the mod is available on multiple Minecraft mod loaders.
The Minecraft version can be found in `gradle.properties`.
Understanding this architecture is crucial when making changes:

### Directory Structure

- **`loader-common/`**: Contains source code that is common across all mod loaders. Most shared functionality should be implemented here.
- **`loader-fabric/`**: Fabric-specific implementation and integration code.
- **`loader-forge/`**: Forge-specific implementation and integration code (for older Minecraft versions).
- **`loader-neoforge/`**: NeoForge-specific implementation and integration code (for newer Minecraft versions).

For NeoForge, relevant jars can for example be found in the following locations:

- Minecraft merged jar: `build/moddev/artifacts/minecraft-patched-26.1.1.1-beta-merged.jar`
- NeoForge: `~/.gradle/caches/modules-2/files-2.1/net.neoforged/neoforge/26.1.1.1-beta/214ad5aa883deaf6d71298baf4232bb994305f36/neoforge-26.1.1.1-beta-universal.jar`
- CyclopsCore source: `~/.m2/repository/org/cyclops/cyclopscore/cyclopscore-26.1.1-neoforge/1.25.5-DEV/cyclopscore-26.1.1-neoforge-1.25.5-DEV-sources.jar`

### Making Changes in Multi-loader Setups

When adding features or fixing bugs:
1. Place shared logic in `loader-common/` whenever possible
2. Only add loader-specific code to the respective `loader-*` directories when platform-specific APIs are required
3. Ensure your changes work across all supported loaders

## Testing

This mod uses two types of tests:

### 1. Unit Tests

**Location**: `src/test/java/`

Traditional JUnit tests for testing isolated functionality without requiring a full Minecraft instance.

**Running unit tests**:
```bash
./gradlew test
```

Unit tests are automatically executed when running the `build` command.

### 2. Game Tests

**Location**: Within normal sources, typically in the `org/cyclops/*/gametest` package (e.g., `loader-common/src/main/java/org/cyclops/cyclopscore/gametest/`)

Game tests run an actual Minecraft instance to test code with real game logic. These are essential for testing features that interact with Minecraft's gameplay systems.
Game tests only exist in Minecraft 1.21 and higher.

**Running game tests**:
```bash
./gradlew runGameTestServer
```

**Important**:
- Game tests are **NOT** run automatically during the build process
- For Minecraft 1.21 and above, game tests must be run manually before committing
- Game tests must pass before finalizing your changes

### When to Add Tests

When adding new features or fixing bugs:
- **Always** add unit tests when possible for isolated logic
- **Always** add game tests when the feature interacts with Minecraft gameplay systems
- Look at existing tests in the respective directories for examples of test patterns and conventions

### Testing advancements

When writing game tests for advancements, don't just call criterion triggers directly, but try to simulate actual game logic to invoke the criterion triggers indirectly.

### Test coverage

When adding new tests for the sake of increasing test coverage, you can measure coverage by running `./gradlew test runGameTestServer jacocoTestReport`, and checking the coverage output in `build/reports/jacoco/test/`.

## Building the Project

### Prerequisites

- Java version is specified in `gradle.properties` (otherwise, default to version 17)
- Gradle (use the provided wrapper: `./gradlew`)

### Build Command

Before every commit, ensure the project builds successfully:

```bash
./gradlew build
```

This command will:
- Compile all source code
- Run unit tests automatically
- Generate build artifacts

### Full Pre-Commit Validation

Run build:

```bash
./gradlew build
```

Only for Minecraft 1.21 and above, also run game tests:
```bash
./gradlew runGameTestServer
```

Both must pass before committing changes.

## Code Formatting

This project uses Spotless for code formatting:

```bash
./gradlew spotlessApply
```

The pre-commit script in `scripts/pre-commit` automatically formats staged files. Consider setting it up as a Git hook:

```bash
ln -s ../../scripts/pre-commit .git/hooks/pre-commit
```

In general, try to use imports where possible.
So avoid inlining things in like `net.minecraft.world.phys.Vec3` in the code, and instead, just add an import for `net.minecraft.world.phys.Vec3`.

## Development Workflow

1. **Understand the change**: Read the issue/feature request thoroughly
2. **Explore the codebase**: Use tools like `grep` to find relevant code
3. **Make minimal changes**: Focus on the specific issue/feature
4. **Add tests**: Write unit tests and/or game tests as appropriate
5. **Build and test**:
   ```bash
   ./gradlew build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CyclopsMC/IconExporter](https://github.com/CyclopsMC/IconExporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
