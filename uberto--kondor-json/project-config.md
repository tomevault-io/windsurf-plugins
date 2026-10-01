---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Kondor-Json** is a functional Kotlin library for JSON serialization/deserialization without reflection, annotations, or code generation. It uses explicit converters (called JConverters) that define bidirectional mappings between domain classes and JSON representation.

### Core Concepts

- **Converters**: Objects that map domain classes to JSON (e.g., `JProduct` for `Product` class)
- **Profunctors**: The mathematical foundation - each converter is a profunctor mapping types bidirectionally
- **Outcome**: Custom Either monad for error handling (not Kotlin's Result)
- **No Reflection**: All mappings are explicit and compile-time safe
- **JsonNode**: Intermediate representation used during parsing (slower but necessary for polymorphic types)

## Working Agreement (how to work in this repo)

Work proceeds in **self-contained chunks**. A chunk is the smallest coherent unit of work (a feature, a fix, a refactor
step) and is only finished when ALL of these hold:

1. **Tests included and green** — the chunk brings its own tests, developed test-first (see the `test-first` skill), and
   the full suite passes: `./gradlew test -x :kondor-mongo:test` (include mongo tests when touched and Docker is
   available).
2. **Docs updated** — relevant `docs/<module>.md` and README.md reflect any behaviour or API change, in the same chunk.
3. **Changelog updated when relevant** — user-visible changes (new API, fixes, breaking changes, performance) get an
   entry in `CHANGELOG.md`; internal refactors don't.
4. **Reviewed** — the adversarial pre-commit review has run and findings are resolved (see the `pre-commit-review`
   skill).
5. **Committed** — one commit per chunk, but **always ask the user before committing**, showing the proposed message and
   a summary of the review.

These skills are mandatory, not optional:

- `kotlin-functional-code` — whenever writing or changing Kotlin code.
- `test-first` — whenever adding features, fixing bugs, or touching tests (includes an adversarial subagent audit of the
  tests).
- `pre-commit-review` — before every commit (includes an adversarial subagent audit of the diff).

**Hard limits**: never `git push`, never create tags/releases/versions, never publish artifacts — those are the user's
manual actions. Don't touch anything outside this repository directory without explicit permission. Inside the
directory, edit freely without asking.

For long unsupervised sessions: keep looping chunk-by-chunk (implement → test → review → ask to commit → next chunk). If
blocked on a genuine product decision, ask; otherwise pick the option most consistent with the existing design and note
it in the commit message proposal.

## Build System

**Requirements**: Java 21 toolchain (configured in gradle/libs.versions.toml)

### Setup for development on MacOS

If you don't have Java 21 installed:
```bash
brew install openjdk@21
```

If you have multiple Java versions installed, you may need to set JAVA_HOME:

```bash
export JAVA_HOME=/opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk/Contents/Home
```

### Common Commands

```bash
# Run all tests
./gradlew test

# Run tests with detailed output
./gradlew test --info

# Run tests for a specific module
./gradlew :kondor-core:test

# Run a single test class
./gradlew :kondor-core:test --tests "com.ubertob.kondor.json.JValuesTest"

# Run a single test method
./gradlew :kondor-core:test --tests "com.ubertob.kondor.json.JValuesTest.Json String with Unicode"

# Build without tests
./gradlew clean build -x test

# Build everything
./gradlew clean build

# Check version
./gradlew printVersion

# Check kondor-core and kondor-outcome against the Java API of Android 13 (also part of `check`)
./gradlew :kondor-core:animalsnifferMain :kondor-outcome:animalsnifferMain

# Rebuild gradle/android-api-33.signature from the Android SDK platform (downloads it if not in $ANDROID_HOME)
scripts/generate-android-signature.sh
```

### Project Structure

Multi-module Gradle project with these modules:

- **kondor-core**: Core JSON parsing/serialization, converters, and DSL
- **kondor-outcome**: Outcome monad for error handling (Either implementation)
- **kondor-tools**: Testing utilities and converter generator
- **kondor-auto**: Automatic converter generation from data classes
- **kondor-examples**: Usage examples (not part of distribution)
- **kondor-mongo**: MongoDB integration using Kondor converters
- **kondor-jackson**: Jackson compatibility layer

## Architecture

### Converter Hierarchy

1. **JsonConverter<T, JN>**: Base interface for all converters
   - `T`: Domain type being converted
   - `JN`: JsonNode type (JsonNodeObject, JsonNodeArray, etc.)

2. **Base Converter Classes**:
   - `JAny<T>`: For objects (uses JsonNode intermediate, slower but flexible)
   - `JObj<T>`: For objects (direct token parsing, faster)
   - `JArray<T, CT>`: For arrays/lists/collections
   - `JSealed<T>`: For sealed classes with discriminator field
   - Primitive converters: `JString`, `JInt`, `JDouble`, `JBoolean`, etc.

3. **Field Definitions**:
   - Fields defined using `by` delegation in converter objects

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uberto/kondor-json](https://github.com/uberto/kondor-json) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
