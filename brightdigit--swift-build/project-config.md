---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a GitHub Action for building and testing Swift packages across multiple platforms. The action supports Swift Package Manager (SwiftPM) builds, Xcode builds for Apple platforms (iOS, macOS, watchOS, tvOS, visionOS), and Android builds using the Swift Android SDK.

## Project Structure

- `action.yml` - Main GitHub Action definition with input parameters and build/test steps
- `.github/workflows/swift-test.yml` - CI workflow testing the action across different platform configurations
- `test/SingleTargetPackage/` - Test Swift package with a single target for validation
- `test/MultiTargetPackage/` - Test Swift package with multiple targets (Core and Utils) for validation

## Build Commands

### Local Swift Package Testing
For testing Swift packages locally:
```bash
# In test/SingleTargetPackage or test/MultiTargetPackage
swift build --build-tests --cache-path .cache --force-resolved-versions
swift test --enable-code-coverage --cache-path .cache --force-resolved-versions
```

### Xcode Testing (macOS only)
For Apple platform testing:
```bash
# macOS
xcodebuild test -scheme [SCHEME] -sdk macosx -destination 'platform=macOS' -enableCodeCoverage YES

# iOS Simulator
xcodebuild test -scheme [SCHEME] -sdk iphonesimulator -destination 'platform=iOS Simulator,name=[DEVICE],OS=[VERSION]' -enableCodeCoverage YES

# Similar patterns for watchOS, tvOS, and visionOS
```

### Android Testing
For Android platform testing (requires skiptools/swift-android-action):
```bash
# Android emulator testing (Ubuntu or Intel macOS)
# Handled by skiptools/swift-android-action@v2
# Supports Swift 6.2+, API levels 28+

# ARM macOS runners: build-only mode (no emulator support)
# Use android-run-tests: false for ARM macOS
```

Note: Android builds delegate to skiptools/swift-android-action. See action.yml for full parameter documentation.

### WebAssembly (Wasm) Testing
For WebAssembly platform testing:
```bash
# Wasm builds use Swift Wasm SDK + WasmKit runtime (default, bundled with Swift 6.2.3+)
# Wasmtime available as optional fallback via wasmtime-version parameter
# Supports: wasm32-unknown-wasi and wasm32-unknown-unknown-wasm (embedded)

# Build and test with WasmKit (default - no downloads required)
swift build --build-tests --swift-sdk swift-6.2.3-RELEASE_wasm
wasmkit run .build/swift-6.2.3-RELEASE_wasm/debug/MyPackageTests.wasm --testing-library swift-testing

# OPTIONAL: Wasmtime fallback for older Swift versions
# Configure via wasmtime-version parameter (e.g., '27.0.0', '26.0.0')
# Breaking Change (v2.0): 'latest' is no longer supported - use specific versions
# Wasmtime binary is automatically cached to avoid ~500MB download per run
# First run with Wasmtime: downloads binary (~3-5 minutes)
# Subsequent runs: uses cached binary (<5 seconds)
```

**WasmKit (Default):**
- Bundled with Swift 6.2.3+ toolchains - no external downloads
- Instant test execution (no download delays)
- No caching overhead (~500MB saved compared to Wasmtime)

**Wasmtime (Fallback):**
- Used when `wasmtime-version` parameter is explicitly specified
- Wasmtime binaries are cached per version to avoid repeated downloads
- Cache key: `wasmtime-{version}-{os}-{arch}`

**Testing Framework Detection (NEW):**
The action automatically detects which testing framework your Wasm tests use:
- **Swift Testing** (`import Testing`) → runs with `--testing-library swift-testing`
- **XCTest** (`import XCTest`) → runs without testing library flag
- **Both frameworks** → runs tests twice (once for each framework)

Override auto-detection with the `wasm-testing-library` parameter:
```yaml
- uses: YourOrg/swift-build@v2
  with:
    type: wasm
    wasm-testing-library: 'swift-testing'  # Force Swift Testing
    wasm-swift-test-flags: '--parallel'    # Optional test runner flags
```

**Code Coverage:** Wasm builds do NOT support code coverage (neither WasmKit nor Wasmtime provide coverage support). Use the `contains-code-coverage` output to conditionally skip coverage collection for Wasm builds.

## GitHub Action Usage

### Inputs

The action accepts these key inputs:
- `scheme` (required) - The scheme to build and test
- `working-directory` - Directory containing the Swift package (default: '.')
- `type` - Build type for Apple platforms (ios, watchos, visionos, tvos, macos)
- `xcode` - Xcode version path for Apple platforms
- `deviceName` / `osVersion` - Simulator configuration for Apple platforms
- `download-platform` - Whether to download platform if not available
- **Android-specific parameters** (auto-detects Android when any are set):
  - `android-swift-version` - Swift version for Android (default: '6.2')
  - `android-api-level` - Android API level (default: '28')
  - `android-ndk-version` - Android NDK version (requires manual NDK installation)
  - `android-run-tests` - Run tests on emulator (default: true; use false for ARM macOS)
  - `android-swift-build-flags` / `android-swift-test-flags` - Additional build/test flags
  - `android-emulator-boot-timeout` - Emulator timeout in seconds (default: '600')
- **Wasm-specific parameters**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brightdigit/swift-build](https://github.com/brightdigit/swift-build) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
