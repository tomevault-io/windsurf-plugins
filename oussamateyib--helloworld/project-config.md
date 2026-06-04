---
trigger: always_on
description: > Guidance for AI agents (e.g. Copilot, Antigravity, Cursor, Claude) working in this repository.
---

# AGENTS.md

> Guidance for AI agents (e.g. Copilot, Antigravity, Cursor, Claude) working in this repository.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Repository Layout](#2-repository-layout)
3. [Architecture](#3-architecture)
4. [Build System](#4-build-system)
5. [Coding Standards](#5-coding-standards)
6. [Common Tasks](#6-common-tasks)
7. [CI/CD Pipeline](#7-cicd-pipeline)
8. [Important Constraints](#8-important-constraints)

---

## 1. Project Overview

**HelloWorld** is a minimal Android application written entirely in **C** using the [raylib](https://github.com/raysan5/raylib) graphics library. It renders the text *"Hello, world!"* centered on a fullscreen window, with the font size calculated dynamically to fill 80 % of the available screen area.

| Property         | Value                                        |
| ---------------- | -------------------------------------------- |
| Language         | C (standard: C23, extensions off)            |
| Graphics library | raylib (git submodule, `master` branch)      |
| Build system     | Gradle + CMake                               |
| Android SDK      | `compileSdk` 37, `minSdk` 21, `targetSdk` 37 |
| Application ID   | `com.oussamateyib.helloworld`                |
| Version          | 1.1.3 (versionCode 5)                        |
| License          | MIT                                          |

---

## 2. Repository Layout

```plaintext
HelloWorld/
├── .github/
│   ├── ISSUE_TEMPLATE/               # Bug report & feature request templates
│   ├── workflows/
│   │   ├── build.yml                 # CI: build, lint, and upload artifacts
│   │   ├── release.yml               # CD: create GitHub releases
|   |   ├── codeql.yml                # CI: Run static analysis
│   │   └── dependency-submission.yml
│   ├── CODE_OF_CONDUCT.md
│   ├── CONTRIBUTING.md
│   ├── pull_request_template.md
│   └── SECURITY.md
├── app/
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── c/
│       │   ├── CMakeLists.txt  # Native build definition
│       │   ├── main.c          # Application entry point (all logic lives here)
│       │   └── raylib/         # Git submodule — DO NOT edit
│       └── res/                # Android resources (icons, strings, XML rules)
├── gradle/                     # Gradle wrapper and version catalog
├── build.gradle.kts            # Root Gradle build script
├── settings.gradle.kts         # Project name and module declarations
├── gradle.properties           # JVM args, caching, and Android flags
├── renovate.json               # Dependency update automation
├── .gitmodules                 # Submodule declaration for raylib
├── .gitignore
├── .gitattributes
├── gradlew / gradlew.bat       # Gradle wrapper scripts
├── LICENSE
├── README.md
└── AGENTS.md
```

---

## 3. Architecture

The application is **fully native** — there is no Kotlin or Java runtime code (`android:hasCode="false"` in the manifest). The Android framework loads the native shared library (`libmain.so`) directly via `NativeActivity`.

```plaintext
Android NativeActivity
        │
        └── libmain.so   (compiled from app/src/main/c/)
                │
                ├── main.c          ← application logic
                └── libraylib.a     ← statically linked from the raylib submodule
```

### Supported ABIs

`x86`, `x86_64`, `armeabi-v7a`, `arm64-v8a`, `riscv64`

> ABI splits are enabled for APK builds and disabled automatically for AAB builds to avoid conflicts.

---

## 4. Build System

### Prerequisites

| Tool        | Version                      |
| ----------- | ---------------------------- |
| JDK         | 17                           |
| CMake       | ≥ 3.25.0                     |
| Android SDK | Managed automatically by AGP |
| Android NDK | Managed automatically by AGP |

### Gradle tasks

```bash
# Build debug and release APKs + AABs
./gradlew build
./gradlew bundle

# Install on a connected device or emulator
./gradlew installDebug
./gradlew installRelease

# Uninstall
./gradlew uninstallDebug
./gradlew uninstallRelease

# Run lint checks
./gradlew lint lintRelease

# Clean all build outputs
./gradlew clean
```

### CMake flags (set automatically by Gradle)

| Flag                 | Value     | Purpose                            |
| -------------------- | --------- | ---------------------------------- |
| `CMAKE_C_STANDARD`   | `23`      | Enforce C23                        |
| `CMAKE_C_EXTENSIONS` | `OFF`     | Disable compiler extensions        |
| `PLATFORM`           | `Android` | Tell raylib to target Android      |
| `BUILD_EXAMPLES`     | `OFF`     | Skip raylib example builds         |
| `APP_LIB_NAME`       | `main`    | Output library name (`libmain.so`) |

### Release signing

Release builds read signing credentials from the following **environment variables**:

| Variable         | Description                                            |
| ---------------- | ------------------------------------------------------ |
| `STORE_FILE`     | Absolute path to the `.jks` / `.p12` keystore          |
| `STORE_PASSWORD` | Keystore password                                      |
| `KEY_ALIAS`      | Key alias inside the keystore                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OussamaTeyib/HelloWorld](https://github.com/OussamaTeyib/HelloWorld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
