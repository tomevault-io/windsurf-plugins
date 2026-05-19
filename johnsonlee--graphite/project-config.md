---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Workflow

Follow these steps in order when working on a task:

1. **Design** — Explore the codebase, understand the problem, and plan the implementation approach. Use plan mode for non-trivial tasks.
2. **Implement** — Write the code changes. Keep changes focused and avoid over-engineering.
3. **Run tests** — Run `./gradlew check` to verify all tests pass. *(Optional when running as Claude Code on web, since CI will catch failures.)*
4. **Commit** — Always show a summary of changes and wait for explicit user confirmation before committing.
5. **Squash commits and submit PR** — Squash all commits on the branch into a single commit, push, and create a PR via `gh pr create`.
6. **Watch CI build result** — CI posts build results as PR comments on failure. If the build fails, read the failure comment, fix the issue, and repeat from step 3.
7. **Ask user to approve and merge** — Once CI passes, ask the user to review, approve, and merge the PR.
8. **Publish** — NEVER publish via local `./gradlew publish*` commands. ALWAYS publish by creating and pushing a git tag, which triggers GitHub Actions to publish automatically: `git tag vX.Y.Z && git push origin vX.Y.Z`. Before publishing:
   1. Check existing versions: `gh api /users/johnsonlee/packages/maven/io.johnsonlee.graphite.graphite-cli/versions --jq '.[].name' | head -5`
   2. Determine the next version number based on existing versions
   3. Show the user the current latest version and proposed new version for confirmation
9. **Update docs** — After tagging a release, update documentation (README version references, etc.) to reflect the new version. This is a docs-only change — commit and push directly to `main` without a PR.

## Project Overview

Graphite is a graph-based static analysis framework for JVM bytecode. It provides a clean abstraction layer over [SootUp](https://github.com/soot-oss/SootUp) for building custom program analyses.

## Test Coverage Requirements

**Unit test line coverage for the entire project MUST be >= 98%.** When adding or modifying code, ensure sufficient tests are written to maintain this threshold.

## Build Commands

```bash
# Build all modules
./gradlew build

# Build specific module
./gradlew :graphite-core:build

# Run tests
./gradlew check

# Run a specific test class
./gradlew :graphite-sootup:test --tests "io.johnsonlee.graphite.sootup.UseCaseValidationTest"
```

## Module Structure

```
graphite/
├── graphite-core/          # Core framework (zero external dependencies except fastutil)
│   ├── core/               # Node, Edge, TypeDescriptor, MethodDescriptor
│   ├── graph/              # Graph interface, DefaultGraph
│   ├── analysis/           # DataFlowAnalysis
│   ├── query/              # QueryDsl - declarative query API
│   └── input/              # ProjectLoader interface, LoaderConfig
│
├── graphite-sootup/        # SootUp backend + GraphiteExtension SPI
│   └── sootup/             # JavaProjectLoader, SootUpAdapter
│
└── cli/
    ├── find-args/          # Find argument constants CLI
    ├── find-endpoints/     # Find HTTP endpoints CLI
    └── find-dead-code/     # Find dead code CLI
```

## Key Abstractions

| Abstraction | Description |
|-------------|-------------|
| `Node` | Program element: constant, variable, parameter, return value, call site |
| `Edge` | Relationship: dataflow, call, type hierarchy |
| `Graph` | Unified program representation, supports traversal and queries |
| `ProjectLoader` | Interface for loading bytecode into Graph |
| `DataFlowAnalysis` | Backward/forward slice analysis |
| `GraphiteQuery` | Declarative query DSL |

## Usage

```kotlin
// Load bytecode
val graph = JavaProjectLoader(LoaderConfig(
    includePackages = listOf("com.example")
)).load(Path.of("/path/to/app.jar"))

// Query: find constants passed to specific methods
Graphite.from(graph).query {
    findArgumentConstants {
        method {
            declaringClass = "com.example.SomeClass"
            name = "someMethod"
            parameterTypes = listOf("java.lang.Integer")
        }
        argumentIndex = 0
    }
}

// Dataflow: backward slice from a node
val analysis = DataFlowAnalysis(graph)
val result = analysis.backwardSlice(nodeId)
result.constants()  // all constant values that flow to this node
```

## Dependency Management

Dependencies are managed via version catalog (`gradle/libs.versions.toml`).

## Publishing

This project publishes artifacts to GitHub Packages. Both manual and automated publishing are supported.

### Prerequisites

Configure GitHub credentials in `~/.gradle/gradle.properties`:

```properties
gpr.user=your-github-username
gpr.key=your-github-token
```

Token requires `write:packages` permission.

### Manual Publishing

Specify the version via command line property:

```bash
# Publish alpha version
./gradlew clean publishAllPublicationsToGitHubPackagesRepository -Pversion=1.1.1-alpha.1

# Publish release version
./gradlew clean publishAllPublicationsToGitHubPackagesRepository -Pversion=1.1.0
```

### Automated Publishing (GitHub Actions)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnsonlee/graphite](https://github.com/johnsonlee/graphite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
