---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Conflict-Free Replicated Data Type (CRDT) library for Protocol Buffer messages, enabling distributed synchronization across devices without coordination. It provides field-level Last-Write-Wins (LWW) conflict resolution that operates directly on protobuf structures with a parallel version tree, avoiding the data duplication and mapping overhead of traditional CRDT libraries.

**Key Innovation:** Complete separation between version management and business data—the library maintains a parallel tree of version nodes matching the protobuf message structure without duplicating field values, enabling O(1) field access and O(m) space overhead where m = modified field count.

## Module Architecture

The repository is organized into 7 modules with clear separation of concerns:

### Core Modules

- **`resolver/`** - Platform-agnostic CRDT conflict resolution algorithms with zero external dependencies beyond Kotlin stdlib. Contains the core logic for field-level LWW semantics, version tree traversal, map/collection strategies, and tombstone cleanup policies. All resolution logic is shared by both Wire and Protoc implementations.

- **`data/`** - Pure protobuf schema definitions (`*.proto` files in `src/main/proto/`). Defines `VersionNode`, `VersionSequence`, `DistributedDocument`, and `Actors` message structures. This is the single source of truth for all proto schemas. Publishes a proto JAR artifact for consumption by build systems like Bazel.

### Data Generation Modules

- **`wire-data/`** - Wire-generated Kotlin classes from the `data` module schemas. Provides idiomatic Kotlin data classes for Android/Kotlin projects.

- **`protoc-data/`** - Java protobuf class generation from the `data` module schemas. Enables backend services to use the same proto definitions without depending on Wire.

### Platform-Specific Implementations

- **`wire/`** - CRDT resolver for Square Wire protobuf library (Kotlin/Android). Uses `@WireField` annotations for compile-time field metadata, providing zero-reflection overhead. Entry point: `WireCrdtResolverProvider`

- **`protoc/`** - CRDT resolver for standard Google protobuf (Java/backend). Uses `getDescriptor()` for runtime field introspection via descriptors. Entry point: `CrdtMessageResolverProvider`

### Testing Support

- **`fixtures/`** - Shared test fixtures and utilities used across test suites

- **`wire/test/`** - Test-specific Wire message definitions

- **`protoc/test/`** - Test-specific protoc message definitions

## Build System

This repository uses **Gradle 9.2.1 with Kotlin DSL**. All dependencies are managed through Gradle version catalogs defined in `settings.gradle.kts`.

### Common Development Commands

#### Building Modules
```bash
# Build all modules
./gradlew build

# Build specific module
./gradlew :resolver:build
./gradlew :data:build
./gradlew :wire-data:build
./gradlew :wire:build
./gradlew :protoc:build
./gradlew :protoc-data:build

# Clean build
./gradlew clean build
```

#### Running Tests
```bash
# Run all tests
./gradlew test

# Run tests for specific module
./gradlew :resolver:test
./gradlew :wire:test
./gradlew :protoc:test

# Run tests with output
./gradlew test --info
```

#### Proto Schema Compilation
```bash
# Wire compilation for data module
./gradlew :wire-data:generateWireProtos

# Wire compilation for wire-test module
./gradlew :wire-test:generateWireTestProtos

# Protoc compilation for protoc-data module
./gradlew :protoc-data:generateProto

# Clean and regenerate all protos
./gradlew clean :wire-data:generateWireProtos :protoc-data:generateProto
```

#### Publishing to Maven Repository
```bash
# Publish all modules to local Maven repository (~/.m2/repository)
./gradlew publishToMavenLocal

# Publish specific module to local Maven
./gradlew :resolver:publishToMavenLocal
./gradlew :wire:publishToMavenLocal

# Publish to Maven Central (requires credentials)
./gradlew publish

# Dry-run to see what would be published
./gradlew publish --dry-run
```

#### Viewing Dependencies
```bash
# View all project dependencies
./gradlew :resolver:dependencies

# View dependency tree for specific configuration
./gradlew :wire:dependencies --configuration runtimeClasspath
```

#### CI

CI runs via GitHub Actions. See `.github/workflows/` for workflow definitions and `ci/README.md` for documentation.

- **CI build & test**: Runs automatically on push to `master` and on pull requests.
- **Publish**: Triggered by pushing a version tag (`v*`).

### Maven Publishing Configuration

The library publishes 5 artifacts to Maven Central:

| Artifact ID | Module | Description |
|-------------|--------|-------------|
| `crdt-resolver` | resolver | Core algorithms (no protobuf deps) |
| `crdt-data` | data | Wire-generated data classes |
| `crdt-wire` | wire | Wire CRDT implementation |
| `crdt-protoc` | protoc | Protoc CRDT implementation |
| `crdt-protoc-data` | protoc-data | Protoc-generated data classes |

**Group ID:** `co.atoms.lithium.crdt`

#### Maven Central Configuration

Publishing credentials are provided via environment variables (set as GitHub Actions secrets):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atoms-co/lithium-crdt](https://github.com/atoms-co/lithium-crdt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
