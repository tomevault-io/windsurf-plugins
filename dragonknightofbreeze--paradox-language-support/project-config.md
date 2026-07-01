---
trigger: always_on
description: This repository contains **Paradox Language Support**, the IntelliJ IDEA plugin designed for Paradox game mod developers.
---

# Guidance to the Paradox Chronicle

## Overview

This repository contains **Paradox Language Support**, the IntelliJ IDEA plugin designed for Paradox game mod developers.

In narrative level, the plugin's name is **Paradox Chronicle**.
While playing on the double meaning of **Chronicle** and **Prophecy Book**, this book is, indeed, also the guide book, to the paradox universe.

Given the large codebase (~110k lines of Kotlin production code) and high complexity (platform & domain & architecture), make this project a challenging undertaking.
As you embark on this journey, keep serious, cautious, and curious.

## Project quick orientation

### What the plugin supports

- **Paradox Script** (`PARADOX_SCRIPT`) - used for providing game data and writing game logic.
- **Paradox Localisation** (`PARADOX_LOCALISATION`) - used for providing i18n text.
- **Paradox CSV** (`PARADOX_CSV`) - used for describing table data.
- **CWT** (`CWT`, `*.cwt`) - used for writing CWT config files which drive semantics (completion, inspections, navigation, docs, etc.).

In addition to language features, the plugin also includes:

- **Image support** (DDS/TGA) with optional tool-based rendering/conversion.
- **Tool integrations** (e.g. ImageMagick, Tiger lint, Translation plugin).
- **AI integration** (LangChain4j-based) focused on localisation translation/polishing workflows.
- A substantial internal **extension point (EP)** architecture and a **code injection** subsystem.

### Key points

- The plugin is written in Kotlin and PSI-based (not LSP-based).
- Many language features are powered by the **config system** based on **CWT config files**.
- The relationship between Paradox script files and CWT config files is roughly like **JSON vs JSON Schema**.

### Project structure (high level)

- `agents/`: agent workflow files (commands, context, rules, skills, etc.).
- `cwt/`: CWT config directories (core + per-game repositories).
- `docs/`: reference documentation (including language syntax guidance and config format guidance).
- `documents/`: maintainer documentation (including ai-generated docs and maintainer-written docs).
- `src/main/kotlin`, `src/main/java`, `src/main/resources`: plugin source.
- `src/test/kotlin`, `src/test/java`, `src/test/resources`: test codes and test resources.
- `src/test/testData/`: test data files (e.g., test-specific cwt config files and script files).
- `src/main/resources/META-INF/plugin.xml`: plugin entry (registrations are split into `META-INF/*.xml`).
- `src/main/resources/META-INF/*.xml`: plugin registrations.

## Setup and build commands (Windows)

This project uses **Gradle** and the **IntelliJ Platform Gradle Plugin**.

### Requirements

- **JDK 21** (the build uses `kotlin.jvmToolchain(21)`)
- Gradle wrapper (use `./gradlew` / `./gradlew` on Windows)

### Common commands

> On Windows PowerShell, prefer running Gradle via the wrapper:`./gradlew <task>`

- Run IDE for debugging: `./gradlew runIde`
- Build the plugin ZIP: `./gradlew buildPlugin`
- Run tests: `./gradlew test`

### CWT config repositories

The local config repos used are in the `cwt/<repoDir>` directory, and there are also some useful scripts in the `cwt/scripts` directory.

The plugin bundles CWT configs into the plugin JAR under `config/<gameTypeId>`.
If missing (common in CI), Gradle can download ZIPs and unzip them into `build/generated/cwt/<repoDir>`.

## Testing guidance

### Test taxonomy

- Prefer Kotlin for tests.
- Unit tests: for pure components/tools/extensions; usually no IntelliJ API.
- Integration tests: for PSI/index/query/semantic match and resolve/tool integrations; usually uses the IntelliJ test framework.
- Tooling: JUnit4 + IntelliJ test framework.

### Test data conventions

- Most platform/integration tests use test data under `src/test/testData`.
- Naming convention is typically `*.test.txt` / `*.test.yml` / `*.test.cwt` / `*.test.csv`.
- Case convention is typically `snake_case.test.txt`.
- Some feature tests provide a test-local `.config/` directory under test data to simulate config groups.

### IntelliJ platform test patterns

- **Parsing tests** (syntax/PSI snapshots): use `ParsingTestCase` and compare the parsed tree output.
- **Fixture-based tests**: use `BasePlatformTestCase` + `myFixture.configureByFile(...)`.

### Config-driven integration tests (config groups + context injection)

The plugin is config-driven. Many features (e.g. type inference, scope inference, macros) depend on **CWT config groups** and a simulated “game/mod context”.

Scope extensions exist to make these tests deterministic:

- `initConfigGroups(project, ...)` initializes the required built-in config groups for the specified game types.
- `markIntegrationTest()` and `clearIntegrationTest()` toggles integration-test-only behavior and cleans up injected state.
- `markRootDirectory(...)` allow integration tests to inject root directory path.
- `markConfigDirectory(...)` allow integration tests to inject config directory path.
- `markFileInfo(...)` and `VirtualFile.injectFileInfo(...)` allow integration tests to inject per-file metadata (game type, logical path, etc.) without requiring a real game installation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DragonKnightOfBreeze/Paradox-Language-Support](https://github.com/DragonKnightOfBreeze/Paradox-Language-Support) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
