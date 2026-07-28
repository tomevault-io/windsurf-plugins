---
trigger: always_on
description: This file provides comprehensive guidance for AI agents and developers working with the Sentry Unity SDK repository.
---

# Agents.md

This file provides comprehensive guidance for AI agents and developers working with the Sentry Unity SDK repository.

---

## 1. Overview & Quick Reference

### Repository Purpose

The Sentry Unity SDK provides error monitoring, performance tracing, and crash reporting for Unity applications across all platforms (Android, iOS, macOS, Windows, Linux, WebGL, PlayStation, Xbox).

### Quick Commands

**IMPORTANT**: Always run `dotnet build` from the repository root. Never build specific `.csproj` files directly.

```bash
# Download prebuilt native SDKs
dotnet msbuild /t:DownloadNativeSDKs src/Sentry.Unity

# Build the Unity SDK (always from root, never target specific .csproj files)
dotnet build

# Run all tests (builds SDK first)
pwsh scripts/run-tests.ps1

# Run specific test types
pwsh scripts/run-tests.ps1 -PlayMode
pwsh scripts/run-tests.ps1 -EditMode

# Run filtered tests
pwsh scripts/run-tests.ps1 -Filter "TestClassName"
pwsh scripts/run-tests.ps1 -PlayMode -Filter "Throttler"

# Skip build for faster iteration
pwsh scripts/run-tests.ps1 -SkipBuild -Filter "MyTest"

# Integration testing (local)
./test/Scripts.Integration.Test/integration-test.ps1 -Platform "macOS" -UnityVersion "2021.3.45f2"

# Create release package
dotnet msbuild /t:DownloadNativeSDKs src/Sentry.Unity
dotnet build
pwsh scripts/repack.ps1
```

### Key Directories

| Directory            | Purpose                                        |
| -------------------- | ---------------------------------------------- |
| `src/`               | Source code for all platform implementations   |
| `package-dev/`       | Development Unity package with all assemblies  |
| `package/`           | Release package template for UPM               |
| `test/`              | Test suite (unit, integration, platform tests) |
| `modules/`           | Git submodules for native SDKs                 |
| `samples/`           | Sample Unity projects                          |
| `scripts/`           | Build automation and testing scripts           |
| `.github/workflows/` | CI/CD workflow definitions                     |

### Git Commit Guidelines

- Use simple, direct commit messages without prefixes like "chore:" or "feat:"
- Messages start with a capital letter

---

## 2. Project Architecture

### Core Components

```
src/
├── Sentry.Unity/           # Main SDK - Unity-specific functionality
├── Sentry.Unity.Editor/    # Editor integration, config windows, build hooks
├── Sentry.Unity.Android/   # Android JNI bridge to sentry-java
├── Sentry.Unity.iOS/       # iOS/macOS Objective-C bridge to sentry-cocoa
└── Sentry.Unity.Native/    # Windows/Linux P/Invoke to sentry-native
```

### Platform Integration Pattern

Each platform follows a consistent architecture:

1. **Native Bridge** - Platform-specific interface to native SDK
   - Android: JNI via `AndroidJavaClass`/`AndroidJavaObject`
   - iOS/macOS: Objective-C via `DllImport("__Internal")`
   - Windows/Linux: P/Invoke via `DllImport("sentry")`

2. **Context Writer** - Synchronizes Unity context (device, GPU, app info) to native SDK

3. **Scope Observer** - Keeps scope (breadcrumbs, tags, user) synchronized between C# and native layers

4. **Configuration** - Platform-specific options and initialization logic

### Assembly Structure

- Runtime assemblies separate from Editor assemblies
- Platform-specific assemblies compile only for target platforms
- Clear dependency hierarchy prevents circular references
- Assembly aliasing prevents symbol conflicts with user dependencies

---

## 3. CI/CD System

### Workflow Architecture

The CI system uses modular, reusable workflows in `.github/workflows/`:

| Workflow                  | Purpose                                            |
| ------------------------- | -------------------------------------------------- |
| `ci.yml`                  | Main pipeline - triggers on push/PR                |
| `build.yml`               | Reusable build workflow                            |
| `sdk.yml`                 | Native SDK builds (Android, Linux, Windows, Cocoa) |
| `test-create.yml`         | Creates integration test projects                  |
| `test-build-android.yml`  | Builds Android test apps                           |
| `test-run-android.yml`    | Runs Android tests on emulator                     |
| `test-build-ios.yml`      | Builds iOS test apps                               |
| `test-compile-ios.yml`    | Compiles iOS Xcode projects                        |
| `test-run-ios.yml`        | Runs iOS tests on simulator                        |
| `release.yml`             | Manual release preparation                         |
| `update-deps.yml`         | Scheduled dependency updates (daily)               |
| `create-unity-matrix.yml` | Generates test matrix                              |

### Unity Version Matrix

| Version  | PR Testing | Main Branch |
| -------- | ---------- | ----------- |
| 2021.3.x | No         | Yes         |
| 2022.3.x | Yes        | Yes         |
| 6000.0.x | Yes        | Yes         |
| 6000.1.x | No         | Yes         |

Version mapping is defined in `scripts/unity-versions.json` (each entry has a `version` and a `changeset`):

- `2021.3` → `2021.3.45f2`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getsentry/sentry-unity](https://github.com/getsentry/sentry-unity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
