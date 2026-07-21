---
trigger: always_on
description: > **Note on Document Formatting:** This document (`AGENTS.md`) should be
---

# Introduction

> **Note on Document Formatting:** This document (`AGENTS.md`) should be
> maintained with lines word-wrapped to a maximum of 80 characters to ensure
> readability across various editors and terminals.

This document provides context and guidance for AI agents (like Jules) when
making changes to the Firebase C++ SDK repository. It covers essential
information about the repository's structure, setup, testing procedures, API
surface, best practices, and common coding patterns.

For a detailed view of which Firebase products are supported on each C++
platform (Android, iOS, tvOS, macOS, Windows, Linux), refer to the official
[Firebase library support by platform table](https://firebase.google.com/docs/cpp/learn-more#library-support-by-platform).

The Firebase C++ SDKs for desktop platforms (Windows, Linux, macOS) are
entirely open source and hosted in the main `firebase/firebase-cpp-sdk` GitHub
repository. The C++ SDKs for mobile platforms (iOS, tvOS, Android) are built
on top of the respective native open-source Firebase SDKs (Firebase iOS SDK and
Firebase Android SDK).

The goal is to enable agents to understand the existing conventions and
contribute effectively to the codebase.

# Setup Commands

## Prerequisites

Before building the Firebase C++ SDK, ensure the following prerequisites are
installed. Refer to the main `README.md` for detailed installation
instructions for your specific platform.

*   **CMake**: Version 3.7 or newer.
*   **Python**: Version 3.7 or newer.
*   **Abseil-py**: Python package.
*   **OpenSSL**: Required for desktop builds, unless you build with the
    `-DFIREBASE_USE_BORINGSSL=YES` cmake flag.
*   **libsecret-1-dev**: (Linux Desktop) Required for secure credential storage.
    Install using `sudo apt-get install libsecret-1-dev`.
*   **Android SDK & NDK**: Required for building Android libraries. `sdkmanager`
    can be used for installation. CMake for Android (version 3.10.2
    recommended) is also needed.
*   **(Windows Only) Strings**: From Microsoft Sysinternals, required for
    Android builds on Windows.
*   **Cocoapods**: Required for building iOS or tvOS libraries.

To build for Desktop, you can install prerequisites by running the following
script in the root of the repository: `scripts/gha/install_prereqs_desktop.py`

To build for Android, you can install prerequisites by running the following
script in the root of the repository: `build_scripts/android/install_prereqs.sh`

## Building the SDK

The SDK uses CMake for C++ compilation and Gradle for Android-specific parts.

### CMake (Desktop, iOS, tvOS)

1.  Create a build directory (e.g., `mkdir desktop_build && cd desktop_build`).
2.  Run CMake to configure: `cmake ..`
    *   For Desktop: Run as is. You can use BORINGSSL instead of OpenSSL (for fewer
        system dependencies with the `-DFIREBASE_USE_BORINGSSL=YES` parameter.
    *   For iOS, include the `-DCMAKE_SYSTEM_NAME=iOS`
        parameter. This requires running on a Mac build machine.
3.  Build specific targets: `cmake --build . --target firebase_analytics`
    (replace `firebase_analytics` with the desired library).
    Or omit the entire `--target` parameter to build all targets.

    For development, building specific targets
    (e.g., `cmake --build . --target firebase_app`) is generally faster and
    recommended once CMake configuration is complete. The full build
    (`cmake --build .`) can be very time-consuming (but can be sped up by adding
    `-j4` to the command-line).

You can also use the `scripts/gha/build_desktop.py` script to build the full
desktop SDK.

Refer to `README.md` for details on CMake generators and providing custom
third-party dependency locations.

### Gradle (Android)

Each Firebase C++ library is a Gradle subproject. To build a specific library
(e.g., Analytics):

```bash
./gradlew :analytics:assembleRelease
```

This command should be run from the root of the repository. Proguard files are
generated in each library's build directory (e.g.,
`analytics/build/analytics.pro`).

You can build the entire SDK for Android by running `./gradlew build` or
`build_scripts/android/build.sh`.

### Xcode (iOS)

Unfortunately, the iOS version of the SDK cannot be built on Linux, it can only
be built in a MacOS environment. You will have to rely on GitHub Actions to
build for iOS, and have the user inform you of any build issues that come up.

### Troubleshooting Desktop Builds

*   Linux: **Missing `libsecret-1-dev`**:
    CMake configuration may fail if `libsecret-1-dev` is not installed.
    The `scripts/gha/install_prereqs_desktop.py` script should handle this.
    If it doesn't, or if the package is removed, you might need to install it
    manually: `sudo apt-get update && sudo apt-get install -y libsecret-1-dev`.

*   Linux: **LevelDB Patch Failure when building Firestore**:
    If you are building the SDK with Firestore enabled
    (`-DFIREBASE_INCLUDE_FIRESTORE=ON`, which is the default for desktop) and
    encounter a patch error related to `leveldb-1.23_windows_paths.patch` (e.g.,
    `util/env_windows.cc: patch does not apply`), you can ignore this issue if
    it does not prevent the rest of the build from running. The patch is only
    important on Windows.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firebase/firebase-cpp-sdk](https://github.com/firebase/firebase-cpp-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
