---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**FTA (Fast Text Analyzer)** is a Java library for semantic type detection and data profiling. It identifies both base types (Boolean, Double, Long, LocalDate, etc.) and semantic types (Email, Phone, SSN, Gender, Country, etc.) from text data. It supports 200+ semantic types across 15+ languages and ~750 locales for date detection.

**Maven coordinates**: `com.cobber.fta:fta` (full) or `com.cobber.fta:fta-core` (base types only)

## Build Commands

```bash
# Build and install distribution
./gradlew clean build installDist

# Run tests with coverage
./gradlew test jacocoTestReport

# Run tests for a specific module
./gradlew :types:test
./gradlew :core:test

# Run a single test class
./gradlew :types:test --tests "com.cobber.fta.TestDates"

# Run a single test method
./gradlew :types:test --tests "com.cobber.fta.TestDates.methodName"

# Generate JavaDoc
./gradlew javadoc

# Build and run examples
./gradlew examples.clean examples.build examples.run

# Check dependency updates
./gradlew dependencyUpdates
```

**Code quality**: SpotBugs (static analysis) and Checkstyle (config in `config/checkstyle/`) are configured. Test framework is TestNG. Coverage via Jacoco (reports in `build/reports/jacoco/test/`).

**Java target**: Java 17 (`options.release = 17`), but the runtime JVM must be **exactly Java 21** — DataSketches 6.2.0 requires it, and the build explicitly rejects JDK 22+. All build and test commands must be prefixed with `JAVA_HOME=/Library/Java/JavaVirtualMachines/amazon-corretto-21.jdk/Contents/Home`, e.g.:

```bash
JAVA_HOME=/Library/Java/JavaVirtualMachines/amazon-corretto-21.jdk/Contents/Home ./gradlew test
```

**Gradle version**: The wrapper targets Gradle 9.5.1 (set in `build.gradle` → `wrapper { gradleVersion = '9.5.1' }`).

**Debugging**: Capture analysis traces with `export FTA_TRACE="enabled=true,directory=/tmp,samples=10000"`, then replay with `cli/build/install/fta/bin/cli --replay <Stream>.fta`.

## Regression Suite (semantic-types)

A golden-file regression suite lives in `../Semantic-types` (separate repo: https://github.com/tsegall/semantic-types). It runs the installed CLI over ~14K real-world CSV files and compares the detected Base Type, Type Modifier, and Semantic Type for every field against a curated baseline (`reference.csv`).

**Before committing any detection-affecting change** — plugins (Java or `plugins.json`), priorities/thresholds, `TypeDeterminer`/`TextAnalyzer` detection logic, date parsing, or reference data — suggest running the suite. Unit tests passing is necessary but not sufficient for these changes. The suite takes several minutes and requires a current `./gradlew installDist`:

```bash
cd ../Semantic-types && bin/runSuiteM.sh   # results land in ncurrent.csv; diff against reference.csv
```

Doc-only, test-only, or serialization-format-neutral changes do not need a suite run.

## Module Structure

```
core/             - Base type detection + date/time parsing (published as fta-core)
types/            - Semantic type detection, profiling, plugin system (published as fta)
cli/              - Command-line interface (Driver.java entry point)
examples/         - Standalone example projects (included builds)
examples/webNG/   - Spring Boot + Vue 3 web UI (included in examples.build; see its AGENTS.md)
```


### core module (`com.cobber.fta.*`)
- `dates/DateTimeParser` — Format detection and parsing across ~750 locales
- `token/` — Pattern/token-based text analysis (`Token`, `SimpleToken`, `FloatToken`, etc.)
- `core/FTAType` — Base type enum (Null, Boolean, Long, Double, LocalDate, LocalTime, etc.)

### types module (`com.cobber.fta.*`)
- `TextAnalyzer` — Single-column streaming/bulk analysis
- `RecordAnalyzer` — Multi-column record-mode analysis with cross-column context
- `Facts` — Profiling metrics (cardinality, patterns, min/max, histogram, signature)
- `LogicalType` / `LogicalTypeFactory` — Base classes for semantic types
  - `LogicalTypeRegExp` — Regex-based types
  - `LogicalTypeFinite` / `LogicalTypeFiniteSimple` — List/finite-set types
  - `LogicalTypeInfinite` — Infinite-set types
- `Plugins` — Plugin manager; loads from `types/src/main/resources/reference/plugins.json`
- `plugins/` — 57+ built-in semantic type implementations (Email, Gender, FirstName, GUID, etc.)
  - `plugins/address/` — Address component types
  - `plugins/identity/` — Identity document types (SSN variants, VAT, etc.)
- `types/src/main/resources/reference/` — 140+ CSV reference data files (countries, postal codes, etc.)

### cli module (`com.cobber.fta.driver.*`)
- `Driver` — Main entry point with argument parsing
- `FileProcessor` — CSV file processing
- `faker/` — Data generation utilities

## Three Analysis Modes

- **Streaming**: `TextAnalyzer.train(String)` — one value at a time; biased by early values
- **Bulk**: `TextAnalyzer.trainBulk(HashMap<String,Long>)` — pre-aggregated counts; faster, unbiased
- **Record**: `RecordAnalyzer.train(String[])` — multi-column with cross-column context biasing

## Plugin Architecture

Two plugin types:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tsegall/fta](https://github.com/tsegall/fta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
