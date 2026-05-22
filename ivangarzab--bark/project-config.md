---
trigger: always_on
description: **Last updated:** 2026-01-27
---

# CLAUDE.md - barK Project Guide

**Last updated:** 2026-01-27
**Purpose:** Quick reference for Claude Code when working with the barK logging library

---

## Project Overview

**barK** is a lightweight, extensible logging library for Kotlin Multiplatform with automatic tag detection and smart test environment handling. Published on Maven Central as `com.ivangarzab.bark:bark`.

**Key Features:**
- Cross-platform support (Android + iOS with platform parity)
- Automatic tag detection from stack traces
- Smart test environment detection
- Multiple concurrent log handlers ("Trainers")
- Runtime control (muzzle/unmuzzle)
- Colored console output for tests
- Dog-themed API

**Philosophy:** SDK-friendly, minimal dependencies, extensible by design.

---

## Project Structure

```
barK/
├── shared/                    # Main KMP library (THIS IS THE LIBRARY)
│   ├── src/
│   │   ├── commonMain/       # Platform-agnostic API
│   │   ├── androidMain/      # Android implementations
│   │   ├── iosMain/          # iOS implementations
│   │   ├── commonTest/       # Shared tests
│   │   ├── androidUnitTest/  # Android tests (18 files)
│   │   └── iosTest/          # iOS tests (6 files)
│   └── build.gradle.kts      # Library build & publishing
├── ios/                       # iOS distribution (BarkExtensions.swift)
├── sample-android/            # Android demo app
├── sample-ios/                # iOS demo app (12 Swift files + 3 test files)
├── tools/                     # Release automation scripts
└── .github/workflows/         # CI/CD pipelines
```

**Important:**
- Library code is in `shared/`, not root
- Sample apps are demos only (not published)
- `/ios` contains BarkExtensions.swift for cleaner Swift API (user-copied file)

---

## Core Architecture

### The Bark Object

Singleton API entry point at `shared/src/commonMain/.../Bark.kt`:
- **Logging:** `v()`, `d()`, `i()`, `w()`, `e()`
- **Trainers:** `train()`, `untrain()`, `releaseAllTrainers()`
- **Control:** `muzzle()`, `unmuzzle()`
- **Tags:** `tag()`, `untag()`
- **Status:** `getStatus()`

### Trainer System (Strategy Pattern)

**Trainer interface** defines custom log handlers with `pack` (categorization) and `minLevel` (filtering).

**Platform-Specific Trainers:**

*Android:*
- `AndroidLogTrainer` - Logcat output (SYSTEM)
- `AndroidTestLogTrainer` - Logcat for tests (SYSTEM)
- `UnitTestTrainer` - Console output (CONSOLE)
- `ColoredUnitTestTrainer` - ANSI colored console (CONSOLE)

*iOS:*
- `NSLogTrainer` - NSLog system logging (SYSTEM)
- `UnitTestTrainer` - Console with timestamps (CONSOLE)
- `ColoredUnitTestTrainer` - ANSI colored with auto-detection (CONSOLE)

**Pack Enum:** CONSOLE, SYSTEM, FILE, CUSTOM. Prevents duplicates except CUSTOM.

### Auto-Detection Features

**Tag Detection (expect/actual):**
- Android: Stack trace parsing via `Thread.currentThread().stackTrace`
- iOS: Symbol parsing for Swift mangled names + Kotlin symbols via `backtrace()`
- Android: 23-char tag limit enforced
- iOS: Auto-detection disabled by default (performance)
- Config: `BarkConfig.autoTagDisabled` (iOS only)

**Test Detection:**
- Android: Detects JUnit, Robolectric, Espresso, AndroidX Test
- iOS: Detects XCTest framework
- Automatically switches between system and console output
- No build-time configuration needed

**Color Support Detection (iOS):**
- Detects ANSI support via `isatty()` and `TERM` env var
- Works in Terminal/CI, not in Xcode test console
- Location: `shared/src/iosMain/.../detectors/ColorSupportDetector.kt`

---

## Common Tasks

### Creating a Custom Trainer
1. Implement `Trainer` interface with `pack` and `minLevel`
2. Add tests in appropriate platform test folder
3. Document in README.md

Reference: `AndroidLogTrainer.kt` or `NSLogTrainer.kt`

### Modifying Tag Detection
- Android: `shared/src/androidMain/.../detectors/TagDetection.kt`
- iOS: `shared/src/iosMain/.../detectors/TagDetection.kt`
- Tests: Check corresponding `TagDetectionTest.kt` files

### Adding Platform-Specific Features
Use expect/actual pattern:
1. Declare `expect` in `commonMain`
2. Implement `actual` in `androidMain` and `iosMain`

### Running Tests
```bash
./gradlew test                          # All tests
./gradlew :shared:test                  # Library only
./gradlew :shared:testDebugUnitTest     # Android
./gradlew shared:iosSimulatorArm64Test  # iOS (Kotlin)
./gradlew shared:koverXmlReport         # Coverage
```

**Coverage:** 90% minimum enforced by Kover.

---

## Development Workflow

### Git Flow
- `main` - Production releases (protected)
- `feature/*` - Feature development
- `release/*` - Release preparation
- `chore/*` - Maintenance

### Making Changes
1. Create feature branch from `main`
2. Make changes in `shared/src/`
3. Write tests (90% coverage required)
4. Update README.md if API changes
5. Run `./gradlew build test`
6. Use conventional commits
7. Create PR to `main`

### Release Process (Maintainer Only)
```bash
./tools/release-process.sh <version>  # Manual
git tag v0.2.0 && git push origin v0.2.0  # Automated (CI/CD)
```

CI/CD: `.github/workflows/prep-release.yml` publishes to Maven Central

---

## Important Files

### Library Core
- `shared/src/commonMain/.../Bark.kt` - Main API
- `shared/src/commonMain/.../Trainer.kt` - Handler interface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivangarzab/barK](https://github.com/ivangarzab/barK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
