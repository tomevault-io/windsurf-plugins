---
trigger: always_on
description: YouTrackDB is a general-purpose object-oriented graph database developed by JetBrains, used internally in production. It implements the Apache TinkerPop API with Gremlin query language support and features O(1) link traversal, schema-less/mixed/full modes, and encryption at rest. The project is a fork of OrientDB, re-architected under the `com.jetbrains.youtrackdb` package namespace.
---

# CLAUDE.md - YouTrackDB Project Guide

## Project Overview

YouTrackDB is a general-purpose object-oriented graph database developed by JetBrains, used internally in production. It implements the Apache TinkerPop API with Gremlin query language support and features O(1) link traversal, schema-less/mixed/full modes, and encryption at rest. The project is a fork of OrientDB, re-architected under the `com.jetbrains.youtrackdb` package namespace.

- **License**: Apache 2.0
- **JDK**: 21+ required
- **Build**: Maven with Maven Wrapper (`./mvnw`)
- **Group ID**: `io.youtrackdb`
- **Version**: `0.5.0-SNAPSHOT` (CI-friendly: `${revision}${sha1}${changelist}`)
- **Issue tracker**: https://youtrack.jetbrains.com/issues/YTDB (YouTrack project code: `YTDB`)
- **Repository**: https://github.com/JetBrains/youtrackdb

## Build Commands

```bash
# Full build (skip tests for speed)
./mvnw clean package -DskipTests

# Full build with unit tests (in-memory storage, default)
./mvnw clean package

# Full build with unit tests on disk storage (as CI does)
./mvnw clean package -Dyoutrackdb.test.env=ci

# Run integration tests (separate from PR pipeline, used by nightly CI)
./mvnw clean verify -P ci-integration-tests

# Build with Docker images (requires Docker)
./mvnw clean package -P docker-images

# Run a single test class
./mvnw -pl core clean test -Dtest=SomeTestClass

# Run a single test method
./mvnw -pl core clean test -Dtest=SomeTestClass#testMethodName
```

**JVM memory**: `.mvn/jvm.config` sets `-Xmx8192m` for Maven itself. Tests use `-Xms4096m -Xmx4096m` (configurable via `heapSize` property).

**Important**: Tests require numerous `--add-opens` JVM flags for Java module system compatibility. These are configured in each module's `pom.xml` `<argLine>` property — do not remove them.

## Project Documentation

The `docs/` folder contains project documentation. See `docs/README.md` for the index.

## Architecture

Storage engine, Gremlin integration, RID format, generated code pipeline, and the table of key entry-point classes (`YourTracks`, `YouTrackDB`, `ServerMain`, `GlobalConfiguration`, `DiskStorage`, `AbstractStorage`, etc.) are documented in `.claude/docs/architecture.md`. Load on demand when the change actually touches storage / Gremlin / parser / generated-code areas.

## Code Style

- **Indent**: 2 spaces (Java, XML, JSON, etc.)
- **Continuation indent**: 4 spaces
- **Line width**: 100 characters
- **Braces**: Always required for `if`, `while`, `for`, `do-while` (force braces = always)
- **Imports**: No wildcard imports (threshold set to 999); import order: static imports first, then regular imports (enforced by Spotless)
- **Wrapping**: Wrap if long for parameters, extends, throws, method chains, binary/ternary operations
- **Binary operators**: Sign on next line when wrapping
- **Blank lines**: 1 blank line after class header, max 1 blank line in code

### Comments and Documentation
- **Comment non-obvious code**: Add comments to any logic that is not immediately self-evident, so reviewers can easily verify intent without reverse-engineering the code.
- **Test descriptions**: Every test must have a detailed description (in a comment or descriptive method name) explaining what scenario is being tested and what the expected outcome is, so a reviewer can quickly grasp the purpose.
- **Keep comments in sync**: When modifying code, always update the surrounding comments to match the new behavior. Stale or contradictory comments are worse than no comments.

### Formatting (Spotless)

Code formatting is enforced by [Spotless](https://github.com/diffplug/spotless) (`com.diffplug.spotless:spotless-maven-plugin`), which runs the `check` goal automatically during the `process-sources` phase of every build. Builds will fail if formatting violations are found.

- **Formatter**: Eclipse formatter configured in `project-config/eclipse-formatter.xml`
- **Ratchet mode**: Only files changed since the `spotless-baseline` git tag are checked — existing code is not reformatted
- **Import order**: Static imports first (`\#`), then regular imports
- **Excludes**: Generated code (`**/internal/core/sql/parser/**`, `**/generated-sources/**`, `**/generated-test-sources/**`)

```bash
# Check formatting (runs automatically during build)
./mvnw spotless:check

# Auto-fix formatting violations
./mvnw spotless:apply

# Check/fix for a single module
./mvnw -pl core spotless:check
./mvnw -pl core spotless:apply
```

**After modifying code, always run `./mvnw -pl {module} spotless:apply`** before committing to ensure formatting compliance. If the build fails with a Spotless error, run `spotless:apply` to auto-fix.

## Writing Style for Design Docs and Issues


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JetBrains/youtrackdb](https://github.com/JetBrains/youtrackdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
