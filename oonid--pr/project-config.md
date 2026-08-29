---
trigger: always_on
description: Guide for AI coding agents working on this project.
---

# AGENTS.md

Guide for AI coding agents working on this project.

## Project Overview

**pr** — Android app that runs Linux distributions via proot (ptrace-based root).
- Android package: `id.or.oo.pr`
- proot version: `5.4.0-pr`, fake kernel: `6.17.0-pr`
- Languages: C (proot), Rust (pr-cli, test binary), Kotlin (Android app), POSIX sh (bootstrap)

## Privacy Policy

**NEVER include personal device information in code, comments, commits, or docs.**

Specifically:
- **NO device model numbers** — mask partial: `SM-XXXXX` is OK, full model is not
- **NO marketing names** — mask partial: `Galaxy X` is OK, full product name is not
- **NO device serial numbers** — mask like `ABCD12XXXX` is OK, real serial is not
- **NO adb serials** in commands, scripts, or documentation — use bare `adb`

**OK to include:**
- Manufacturer: Samsung
- Architecture: aarch64, arm64-v8a
- OS version: Android 16 (SDK 36)
- SELinux context: untrusted_app, runas_app

When writing adb commands in docs/scripts, use bare `adb` without `-s <serial>`.

## Directory Structure

```
src/proot/                  # Patched proot C source (working copy, NOT vendor/)
src/pr-cli/                 # Rust CLI (replaces proot-distro.sh)
src/proot-integration-test/ # Guest-side test binary (runs inside proot, TAP output)
src/scripts/                # Shell scripts: bootstrap.sh (POSIX)
android/                    # Android APK (Kotlin + Compose + JNI)
scripts/                    # Host-side build scripts (build.sh, download-*.sh)
vendor/                     # Git submodules — READ ONLY, never modify
docs/                       # Architecture and feature documentation
build/                      # Build artifacts (gitignored)
```

## Build Process

### 1. Build proot (C, NDK cross-compilation)

```bash
scripts/build.sh --arch=arm64
# Output: build/out/arm64/proot, build/out/arm64/loader
# Copy to android/app/src/main/jniLibs/arm64-v8a/libproot.so
# Copy to android/app/src/main/jniLibs/arm64-v8a/libproot-loader.so
```

### 2. Build pr-cli (Rust, NDK cross-compilation)

```bash
# MUST run from src/pr-cli/ (required for .cargo/config.toml resolution)
cd src/pr-cli && cargo build --target aarch64-linux-android --release
# Output: target/aarch64-linux-android/release/pr-cli (~900KB)
# Copy to android/app/src/main/jniLibs/arm64-v8a/libpr-cli.so
```

### 3. Build test binary (guest-side, cross-compiled)

```bash
# MUST run from src/proot-integration-test/
cd src/proot-integration-test && cargo build --target aarch64-linux-android --release
# Embedded into pr-cli via include_bytes! at build time
# Must rebuild pr-cli after rebuilding test binary
```

### 4. Build APK

```bash
cd android && ./gradlew assembleDebug
# Output: android/app/build/outputs/apk/debug/app-debug.apk
```

### 5. Install on device

```bash
adb install -r android/app/build/outputs/apk/debug/app-debug.apk
# User must force-close and reopen app after install
```

### Full rebuild sequence (after changing test binary)

```bash
# 1. Build test binary
cd src/proot-integration-test && cargo build --target aarch64-linux-android --release
# 2. Build pr-cli (embeds test binary)
cd src/pr-cli && cargo build --target aarch64-linux-android --release
# 3. Copy to jniLibs (ALWAYS verify with md5sum)
cp -f src/pr-cli/target/aarch64-linux-android/release/pr-cli \
      android/app/src/main/jniLibs/arm64-v8a/libpr-cli.so
md5sum src/pr-cli/target/aarch64-linux-android/release/pr-cli \
       android/app/src/main/jniLibs/arm64-v8a/libpr-cli.so
# 4. Build APK
cd android && rm -rf app/build && ./gradlew assembleDebug
# 5. Install
adb install -r android/app/build/outputs/apk/debug/app-debug.apk
```

**IMPORTANT:** Gradle caches jniLibs — always verify the copy succeeded with `md5sum` and clean `app/build` before building the APK.

## Testing

### Host-side (pr-cli unit/integration)

```bash
cd src/pr-cli && cargo test
# 32 tests: 12 unit + 20 integration (14 plugin parsers)
```

### On-device integration tests

Run from app UI via BugReport button on each distro row, or:

```bash
adb shell run-as id.or.oo.pr files/usr/bin/pr-cli test alpine
adb shell run-as id.or.oo.pr files/usr/bin/pr-cli test debian
```

**Test suites:** distro (8), clone (5), readlink (6), gcc (3), rust (4), git (3), pipe (3), general (5)

**Results:** 37/37 pass on Alpine and Debian. Rust suite 4/4 on Debian (rustc compile + cargo build work).

### Known test failures (Alpine-specific)

- **rust suite on Alpine**: 1/4 pass (rustc -vV works). rustc compile and cargo build fail with "linker `cc` not found" — `apk add gcc` installs gcc but the `cc` symlink is not found via PATH inside proot. Not a proot bug; Debian's gcc package creates `/usr/bin/cc` correctly.

## Code Conventions

### Working directory requirements

- `src/pr-cli/` — must be cwd for cargo build (`.cargo/config.toml`)
- `src/proot-integration-test/` — must be cwd for cargo build
- `android/` — must be cwd for gradlew

### Vendor directories are READ ONLY

All modifications go in `src/proot/` (working copy), never in `vendor/`.

### Native library disguise

All native binaries (proot, loader, busybox, bash, pr-cli) are named `lib*.so` in jniLibs/ so Android extracts them to nativeLibraryDir where SELinux allows execve. They are standalone ELF executables, not shared libraries.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oonid/pr](https://github.com/oonid/pr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
