---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Behavioral

1. Don’t assume. Don’t hide confusion. Surface tradeoffs.
2. Minimum code that solves the problem. Limit speculative additions.
3. Touch only what you must, clean up only your own mess -- but do suggest additional related fixes.
4. Define success criteria. Loop until verified.

## Project Overview

Jackson Dataformats Binary is a multi-module umbrella project containing Jackson binary
format backends: CBOR, Smile, Avro, Protobuf, and Ion. Each module provides Jackson
streaming (`TokenStreamFactory` / `JsonParser` / `JsonGenerator`) implementations for one
binary format, plus an `ObjectMapper` subtype.

This file documents work on the **3.x line**: package prefix `tools.jackson.dataformat.*`,
group id `tools.jackson.dataformat`, Java 17 baseline. The 2.x line lives on its own
branches, uses `com.fasterxml.jackson.dataformat.*`, and has different conventions —
notably `<Format>Parser.Feature` / `<Format>Generator.Feature` instead of the standalone
feature enums described below.

## Branches: read this before creating any branch or PR

Active branches, newest-first, with the version each carries:

| Branch | Role |
|---|---|
| `2.21` | Oldest maintained 2.x patch line; most bug fixes start here |
| `2.22` | Current 2.x patch line |
| `2.x`  | Next 2.x minor; new 2.x features and API additions |
| `3.1`  | Oldest maintained 3.x patch line; where `2.x` lands on the Jackson 3 side |
| `3.2`  | Current 3.x patch line |
| `3.x`  | Jackson 3 development tip; new features and API additions |

For the version a branch carries, read it off `origin`, not a local ref, which may be
stale: `git show origin/<branch>:pom.xml | head -12`.

Two branches look active but are not:

- **`3.0`** is closed. `2.x` merges into `3.1` now, not `3.0`.
- **`master`** is abandoned despite `origin/HEAD` pointing at it: last touched April 2025,
  still carrying `3.0.0-rc3-SNAPSHOT`. **Never target it.**

### Merge-forward model

Changes flow forward and are never cherry-picked backward:

```
2.21  →  2.22  →  2.x  →  3.1  →  3.2  →  3.x
```

Note that `2.x` merges into `3.1`, the oldest maintained 3.x branch, not directly into `3.x`.

Pick the target branch by what the change is:

- **Bug fix present in 2.x too**: the oldest maintained branch that has the bug — usually
  `2.21`. It gets merged forward from there, and reaches 3.x automatically. Fixing it
  directly on a 3.x branch means the 2.x releases never get it, and the next merge-forward
  may conflict.
- **Bug fix in 3-only code**: the oldest maintained 3.x branch that has the bug — usually `3.1`.
- **New feature, new API, new config flag**: `3.x` (and `2.x` if it should also ship on the
  2 line). Patch branches take fixes only.

If unsure whether something counts as a fix or a feature, ask rather than guessing —
targeting too new a branch means the fix never reaches released versions, and targeting
too old a branch means an unwanted API change ships in a patch release.

### PR branch naming

Recent convention, matching what's on the remote:

```
tatu-claude/<target-branch>/<issue-number>-<short-slug>
```

Examples: `tatu-claude/3.1/669-cbor-string-ref`, `tatu-claude/3.1/700-cbor-mapper-spi`,
`tatu-claude/3.2/22-smile-root-context-index`.

The branch segment must match the branch the PR targets.

## Build and Test Commands

Always use the wrapper (`./mvnw` from root, `../mvnw` from a module dir). Requires JDK 17+.

```bash
# Build all modules
./mvnw clean install

# Build without running tests
./mvnw clean install -DskipTests

# Verify all modules
./mvnw verify
```

Prefer running tests from inside the module directory — a `-Dtest=` filter run from the
root has to match in every module.

```bash
cd cbor && ../mvnw test                                  # whole module
cd cbor && ../mvnw -Dtest=CBORParserTest test            # one class
cd cbor && ../mvnw -Dtest=CBORParserTest#testSimpleArray test   # one method
```

Coverage:

```bash
./mvnw test jacoco:report      # reports land in <module>/target/site/jacoco/
```

## Repository Structure

Each backend (`avro/`, `cbor/`, `ion/`, `protobuf/`, `smile/`) follows the same layout:

```
<format>/
├── src/main/java/
│   ├── module-info.java              # JPMS descriptor: exports + provides
│   └── tools/jackson/dataformat/<format>/
│       ├── <Format>Factory.java          # creates parsers/generators
│       ├── <Format>FactoryBuilder.java   # builder for factory construction
│       ├── <Format>Parser.java           # streaming reader
│       ├── <Format>Generator.java        # streaming writer
│       ├── <Format>Mapper.java           # ObjectMapper subclass; Ion differs, see below
│       ├── <Format>ReadFeature.java      # format-specific parser features
│       ├── <Format>WriteFeature.java     # format-specific generator features
│       └── PackageVersion.java.in        # template; PackageVersion.java is generated at build time
├── src/main/resources/META-INF/services/
│   ├── tools.jackson.core.TokenStreamFactory
│   └── tools.jackson.databind.ObjectMapper
├── src/test/java/...
└── pom.xml
```

Registration is declared **twice** — once in `module-info.java` (`provides ... with ...`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FasterXML/jackson-dataformats-binary](https://github.com/FasterXML/jackson-dataformats-binary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
