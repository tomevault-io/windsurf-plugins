---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is the Bazel for CLion plugin (and historically IntelliJ) repository. The plugin is maintained by JetBrains as of July 2025. It provides Bazel build system integration for JetBrains IDEs.

**Important:** The CLion plugin is built from the `master` branch. The IntelliJ plugin is built from the `ijwb` branch (maintenance-only). The Android Studio plugin is maintained in AOSP.

## Build Commands

### Building the Plugin

Build the plugin for CLion:
```bash
bazel build //clwb:clwb_bazel_zip --define=ij_product=clion-oss-latest-stable
```

The plugin zip will be created at `bazel-bin/clwb/clwb_bazel.zip`.

### Product Version Aliases

The `ij_product` flag controls which IDE version to build for:
- `clion-oss-oldest-stable` - Oldest supported IDE version
- `clion-oss-latest-stable` - Latest supported IDE version
- `clion-oss-under-dev` - Upcoming IDE version being worked on
- Direct versions: `clion-2025.2`, `clion-2025.3`, etc.

Version mappings are defined in `intellij_platform_sdk/build_defs.bzl`.

### Running Tests

**Important:** Always run tests against ALL supported IDE versions to ensure compatibility:
- `clion-oss-oldest-stable`
- `clion-oss-latest-stable`
- `clion-oss-under-dev`

Run unit tests for a specific module:
```bash
bazel test //base:unit_tests --define=ij_product=clion-oss-latest-stable
bazel test //base:unit_tests --define=ij_product=clion-oss-oldest-stable
bazel test //base:unit_tests --define=ij_product=clion-oss-under-dev
```

Run integration tests:
```bash
bazel test //base:integration_tests --define=ij_product=clion-oss-latest-stable
bazel test //base:integration_tests --define=ij_product=clion-oss-oldest-stable
bazel test //base:integration_tests --define=ij_product=clion-oss-under-dev
```

Run all tests for a language plugin:
```bash
bazel test //cpp/... --define=ij_product=clion-oss-latest-stable
bazel test //cpp/... --define=ij_product=clion-oss-oldest-stable
bazel test //cpp/... --define=ij_product=clion-oss-under-dev
```

Run aspect tests:
```bash
bazel test //aspect/... --define=ij_product=clion-oss-latest-stable
bazel test //aspect/... --define=ij_product=clion-oss-oldest-stable
bazel test //aspect/... --define=ij_product=clion-oss-under-dev
```

### Test Structure

Tests are organized into three categories:
- `unittests/` - Fast, isolated unit tests
- `integrationtests/` - Still relatively fast, used for interacting with IntelliJ platform
- `headlesstests/` - Slow, require headless CLion, used for testing the entire sync process 

### Project View Files

`.bazelproject` files control what gets imported. Key sections:
- `directories:` - Which directories to include/exclude (prefix with `-` to exclude)
- `targets:` - Which Bazel targets to build
- `build_flags:` - Bazel flags to use (including `--define=ij_product=...`)
- `workspace_type: intellij_plugin` - Marks this as a plugin project

## Architecture

### Plugin Structure

The plugin is organized as:
- **`/base`** - Core plugin machinery (sync, project import, run configurations, etc.)
- **Language plugins** - `/cpp`, `/python`, `/javascript`, `/skylark`
- **Product bundles** - `/clwb` (CLion), `/ijwb` (IntelliJ, deprecated)
- **Supporting dirs** - `/aspect`, `/sdkcompat`, `/testing`

#### Legacy Layout (Monolithic)

Each language plugin historically followed this structure:
```
language/
├── BUILD                    # Single BUILD file per plugin (600+ lines)
├── src/
│   ├── META-INF/           # Plugin XML config files
│   └── com/google/...      # Source code
└── tests/
    ├── unittests/          # Fast unit tests
    └── integrationtests/   # Tests requiring IDE instance
```

**Examples:** `/base/BUILD` (628 lines), `/cpp/BUILD`, `/python/BUILD`

This monolithic approach has all targets for a plugin in one BUILD file, making it harder to maintain as the plugin grows.

#### Modern Layout (Fine-Grained)

New code follows a more Bazel-like, fine-grained project structure:
```
module/
├── BUILD                                    # Aggregator with exports
├── src/com/google/idea/blaze/
│   ├── submodule1/
│   │   ├── BUILD                           # Individual library
│   │   └── *.java
│   └── submodule2/
│       ├── BUILD                           # Individual library
│       └── *.java
└── tests/unittests/com/google/idea/blaze/
    ├── BUILD                               # Every test file is a Bazel target
    └── *Test.java
```

This approach provides:
- Better dependency visibility and control
- Easier to understand module boundaries
- More parallelizable builds
- Cleaner refactoring paths

### Key Concepts

#### Sync Process

"Syncing" translates the Bazel build into IntelliJ's project model:

1. **Build Phase**: Run `bazel build` with the aspect in `/aspect`
   - The aspect traverses targets and outputs `*.intellij-info.txt` proto files
   - These protos contain information needed by the IDE (generated files, dependencies, etc.)

2. **Update Phase**: Read the protos and:
   - Generate a `TargetMap` (in-memory representation of the Bazel build)
   - Translate into IntelliJ modules and libraries
   - Update project structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bazelbuild/intellij](https://github.com/bazelbuild/intellij) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
