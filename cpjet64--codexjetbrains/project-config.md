---
trigger: always_on
description: This document is for automation agents and humans to build, test, and run the plugin consistently using the IntelliJ Platform Gradle Plugin **2.x** and **JDK 21**.
---

# AGENTS.md — CodexJetbrains

## Purpose
This document is for automation agents and humans to build, test, and run the plugin consistently using the IntelliJ Platform Gradle Plugin **2.x** and **JDK 21**.

---

## Prerequisites
- **JDK 21** (JAVA_HOME should point to a JDK 21 install)
- **Gradle Wrapper** checked in (`./gradlew`)
- Internet access to JetBrains artifact repos (via cache-redirector)

---

## Configuration Rules
- **Repositories live in `settings.gradle.kts`** (pluginManagement + dependencyResolutionManagement).
- The project **must not** declare `repositories {}` in `build.gradle.kts`.
- The IntelliJ Gradle plugin is **`org.jetbrains.intellij.platform`**, version **2.9.0** (resolved via settings).
- Kotlin plugin version: **1.9.24** (declared in `build.gradle.kts`).

---

## Key Build Parameters
- `platformVersion` — baseline IDE version to resolve (default: `2024.2`).
  Set with: `-PplatformVersion=2024.2` or `ORG_GRADLE_PROJECT_platformVersion`.
- `codex_withJava` — include the `com.intellij.java` bundled plugin (default: `false`).
  Set with: `-Pcodex_withJava=true` or `ORG_GRADLE_PROJECT_codex_withJava`.

### Compatibility Guidance
- `codex_withJava=true` → compatible with IDEs that ship Java (IDEA, Android Studio…).
- `codex_withJava=false` → broader IDE coverage (PyCharm/WebStorm/Rider etc.), **only if the code does not use Java PSI**.

---

## Canonical Commands

### Run tests
```bash
./gradlew test
```

### Build plugin distribution
```bash
./gradlew buildPlugin
```

### Verify plugin compatibility
```bash
./gradlew verifyPlugin
```

### Run plugin in sandbox IDE
```bash
./gradlew runIde
```

### Clean build
```bash
./gradlew clean build
```

---

## Test Suite

### Current Status
- **Total tests**: 114
- **All tests passing**: ✅
- Test framework: JUnit4 with kotlin.test and IntelliJ Platform test fixtures

### Key Test Files
- Unit tests for parsing, telemetry, configuration, etc.
- Integration tests:
  - `PatchApplierIntegrationTest` — validates patch application to files
  - `DiagnosticsServiceTest` — validates sensitive data redaction

### Running specific tests
```bash
./gradlew test --tests "dev.curt.codexjb.tooling.PatchApplierIntegrationTest"
```

---

## Run Configurations

Shared run configurations are available in `.idea/runConfigurations/`:
- **Run Plugin** — launches plugin in sandbox IDE
- **Build Plugin** — builds plugin distribution
- **Verify Plugin** — runs plugin verification
- **Run Tests** — executes test suite

---

## Architecture Notes

### Patch Application
- `PatchApplier.apply()` — main entry point for applying unified diffs
- `PatchApplier.doApplyWithBase()` — internal testable function that accepts custom base path
- `PatchEngine.apply()` — low-level patch application logic (unit tested)
- `UnifiedDiffParser.parse()` — parses unified diff format

### Security
- `SensitiveDataRedactor` — redacts API keys and tokens (e.g., `sk-[A-Za-z0-9]{16,}`)
- `DiagnosticsService` — logs with automatic redaction

---

## Common Issues

### Test Failures
- Ensure JDK 21 is used
- Run `./gradlew clean test` to clear cached test results
- Check that no IDE instances are holding file locks

### Build Failures
- Verify internet connectivity for dependency resolution
- Check `~/.gradle/caches` for corrupted artifacts
- Ensure JAVA_HOME points to JDK 21

---

## Version Information
- **IntelliJ Platform Gradle Plugin**: 2.9.0 (2.x series)
- **Target Platform**: 2024.2
- **Kotlin**: 1.9.24
- **JDK**: 21
- **Gradle**: 8.7 (via wrapper)

## Additional Context

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CodexJetbrains is an unofficial IntelliJ Platform plugin that integrates with the Codex CLI. It provides a dedicated ToolWindow for interacting with Codex without leaving the IDE, including message review, approval handling, and diff application.

- **Target Platform**: IntelliJ IDEA 2025.2+ (build 242+)
- **Runtime**: JDK 21 (enforced via Gradle toolchain)
- **Language**: Kotlin 1.9.24
- **Build System**: Gradle with IntelliJ Platform Gradle Plugin 2.9.0

## Build Commands

### Prerequisites
- JDK 21 (`JAVA_HOME` must point to JDK 21)
- Codex CLI available on PATH or configured in plugin settings

### Core Gradle Tasks
```bash
# Build, test, and package the plugin
./gradlew test verifyPlugin buildPlugin

# Run a sandbox IDE for manual testing
./gradlew runIde

# Run only tests
./gradlew test
```

### Platform-Specific Wrappers
The repository includes helper scripts in `scripts/dev/`:

**Windows (PowerShell)**:
- `powershell -ExecutionPolicy Bypass -File scripts/dev/win-build.ps1` - Build with logging
- `powershell -ExecutionPolicy Bypass -File scripts/dev/win-run-ide.ps1` - Run sandbox with logging
- `powershell -ExecutionPolicy Bypass -File scripts/dev/verify.ps1` - Full verification pipeline
- `powershell -ExecutionPolicy Bypass -File scripts/dev/windows-smoke.ps1` - Quick smoke test

**Linux/macOS (Bash)**:
- `bash scripts/dev/unix-build.sh` - Build with logging
- `bash scripts/dev/unix-run-ide.sh` - Run sandbox with logging

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cpjet64/CodexJetbrains](https://github.com/cpjet64/CodexJetbrains) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
