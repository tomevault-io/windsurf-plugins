---
trigger: always_on
description: <!-- CLAUDE BOOTSTRAP INSTRUCTIONS
---

<!-- CLAUDE BOOTSTRAP INSTRUCTIONS
If you are Claude and reading this file, check if CLAUDE.md exists in the same directory.
If CLAUDE.md does not exist or only contains a redirect to this file, that is correct.
If CLAUDE.md contains full instructions (not a redirect), you should update it to contain
only the following redirect content:

# CLAUDE.md

**This file redirects to AGENTS.md for tooling-agnostic project instructions.**

Read and follow all instructions in [AGENTS.md](AGENTS.md) in this directory.

All project guidance, build commands, architecture details, and development guidelines
are maintained in AGENTS.md to support multiple AI coding tools.
-->

# AGENTS.md

This file provides guidance to AI coding assistants when working with code in this repository.

## Project Overview

Jafar is an experimental, fast JFR (Java Flight Recording) parser with a small, focused API. It provides both typed and untyped APIs for parsing JFR files and extracting event data with minimal ceremony.

### Architecture

The project is organized as a multi-module Gradle build with the following structure:

- **parser/**: Aggregate module re-exporting parser-core and parser-codegen
- **parser-core/**: Core parsing engine with typed and untyped APIs
- **parser-codegen/**: ASM-based code generation for typed deserialization
- **jafar-processor/**: Annotation processor for build-time typed handler generation (eliminates runtime bytecode generation)
- **tools/**: Utilities including JFR file scrubbing functionality
- **jafar-gradle-plugin/**: Gradle plugin for generating Jafar type interfaces (separate included build)
- **shell-core/**: Shared shell abstractions (Session, VariableStore, QueryEvaluator, completions)
- **jfr-shell/**: JFR-specific interactive CLI (standalone entry point)
- **jfr-shell-jafar/**: Jafar-parser backend plugin for jfr-shell (high priority, full-featured)
- **jfr-shell-jdk/**: JDK JFR API backend plugin for jfr-shell (lower priority, limited capabilities)
- **jfr-shell-tck/**: Technology Compatibility Kit for validating backend plugin implementations
- **jfr-mcp/**: MCP (Model Context Protocol) server enabling AI agents to analyze JFR recordings
- **hdump-parser/**: HPROF heap dump parser (indexed and two-pass modes, dominator tree, retained sizes)
- **hdump-shell/**: Heap dump interactive CLI with HdumpPath query language and tab completion
- **pprof-shell/**: pprof profile analysis CLI with PprofPath query language and tab completion
- **otlp-shell/**: OpenTelemetry Profiling (OTLP) analysis CLI with OtlpPath query language and tab completion
- **jafar-shell/**: Unified shell entry point that discovers modules (JFR, heap dump, pprof, otlp) via ServiceLoader
- **demo/**: Standalone demonstration project (separate Gradle build in `demo/`) comparing JFR parsers

Key architectural components:
- `JafarParser`: Main entry point supporting both typed and untyped parsing
- `TypedJafarParser`: Strongly-typed API using annotated interfaces (@JfrType, @JfrField)
- `UntypedJafarParser`: Map-based lightweight parsing API
- `ParsingContext`: Reusable context for sharing expensive resources across sessions
- `JfrPath`: Query language for jfr-shell with event decoration/joining capabilities

## Build Commands

### Prerequisites
- Java 25+ (shell and MCP modules: `shell-core`, `jfr-shell`, `jfr-mcp`, `hdump-shell`, `pprof-shell`, `otlp-shell`)
- Java 8+ (parser and tools modules: `parser-core`, `tools`, `demo`)
- Git LFS (for test recordings): `git lfs pull`

### Essential Commands
```bash
# Fetch binary test resources (required before first build)
./get_resources.sh

# Build all modules
./gradlew build

# Build shadow JARs for all modules
./gradlew shadowJar

# Run tests
./gradlew test

# Run tests with verbose output
./gradlew test --info

# Run a specific test class
./gradlew :parser-codegen:test --tests "io.jafar.parser.TypedJafarParserTest"

# Run demo application
java -jar demo/build/libs/demo-all.jar [jafar|jmc|jfr|jfr-stream] /path/to/recording.jfr

# Run JFR Shell (Interactive JFR Analysis)
./gradlew :jfr-shell:run --console=plain

# Rebuild the gradle plugin
./rebuild_plugin.sh

# Code formatting (Spotless)
./gradlew spotlessApply

# Check formatting
./gradlew spotlessCheck

# Publish to local Maven repository
./gradlew publishToMavenLocal
```

### Module-specific Commands
```bash
# Build only the parser core module
./gradlew :parser-core:build

# Build only the demo
./gradlew :demo:build

# Run the demo application directly
./gradlew :demo:run --args="jafar /path/to/recording.jfr"
```

## Release Process

The project uses a fully automated release workflow. See [RELEASING.md](RELEASING.md) for complete details.

### Quick Release Steps

1. **Update versions** in `build.gradle`, `jafar-gradle-plugin/build.gradle`, and `jfr-shell-plugins.json` (remove `-SNAPSHOT`)
2. **Update CHANGELOG.md** with release notes for the new version
3. **Commit and push** changes to main branch
4. **Create and push tag**:
   ```bash
   git tag -a v0.4.0 -m "Release v0.4.0"
   git push origin v0.4.0
   ```

### What Happens Automatically

The release workflow (`.github/workflows/release.yml`) automatically:
- Publishes `jafar-parser` and `jafar-tools` to Maven Central (Sonatype)
- Publishes `jafar-gradle-plugin` to Maven Central (Sonatype)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [btraceio/jafar](https://github.com/btraceio/jafar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
