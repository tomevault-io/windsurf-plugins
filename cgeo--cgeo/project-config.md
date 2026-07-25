---
trigger: always_on
description: This document provides repository-wide instructions for GitHub Copilot agents working on the c:geo project.
---

# c:geo Copilot Instructions

This document provides repository-wide instructions for GitHub Copilot agents working on the c:geo project.

## Sources of Truth

When instructions conflict, apply this priority order (highest first):

1. This file (`/.github/copilot-instructions.md`) — agent behavior rules
2. `/checkstyle.xml` and `/.editorconfig` — authoritative code style and formatting rules
3. `/README.md` — general project and contributor context
4. `/.github/workflows/` — reference for what the real CI pipeline does

Only reference files that are tracked in git (i.e. not excluded by `/.gitignore`).

## About c:geo

c:geo is an open-source Android geocaching app written in Java. It's a full-featured client for geocaching.com (unofficial) and offers basic support for other geocaching platforms.

## Project Structure

- `main/` - Main application module
  - `src/main/java/` - Main application source code
  - `src/test/java` - Pure unit tests (JUnit)
  - `src/androidTest/java` - Android instrumented tests
  - `build.gradle` - Module build configuration
- `checkstyle.xml` - Checkstyle configuration (authoritative style rules)
- `ruleset.xml` - PMD ruleset (code quality rules)
- `suppressions.xml` - Checkstyle suppressions
- `.editorconfig` - Authoritative formatting rules (indentation, import layout)

## Code Style and Quality

The **authoritative rules** for code style are defined in `checkstyle.xml` and `.editorconfig`.
The summary below reflects those files. If in doubt, those files take precedence over this prose.

### Import Organization

Imports must be organized into the following groups, separated by blank lines
(matches `ImportOrder` in `checkstyle.xml` and `ij_java_imports_layout` in `.editorconfig`):

1. `cgeo.*` — c:geo internal packages
2. `android.*` — Android framework
3. `androidx.*` — AndroidX libraries
4. `java.*` — Java standard library
5. `javax.*` — Java extensions
6. `*` — All other imports

Static and non-static imports from the same group must not be separated by a blank line.
Within each group, imports must be sorted alphabetically.

### Code Conventions (Must)

- **No unused imports** — remove all unused imports from modified files (`UnusedImports` in `checkstyle.xml`, severity warning)
- **No star imports** — always use explicit imports (`AvoidStarImport` in `checkstyle.xml`)
- **No tabs** — use spaces for indentation (`FileTabCharacter` in `checkstyle.xml`; `indent_style = space` in `.editorconfig`)
- **4-space indent** — indent size is 4 (`indent_size = 4` in `.editorconfig`)
- **File ends with newline** — all files must end with a newline (`NewlineAtEndOfFile` in `checkstyle.xml`)
- **`@Override` annotation** — always use `@Override` where applicable (`MissingOverride` in `checkstyle.xml`, severity warning)
- **No `equals()` without `hashCode()`** — keep them in sync (`EqualsHashCode` in `checkstyle.xml`, severity warning)
- **`final` for local variables** — prefer `final` for local variables whenever possible (`FinalLocalVariable` in `checkstyle.xml`, severity warning)
- **`final` for parameters** — prefer `final` method parameters whenever possible (`FinalParameters` in `checkstyle.xml`, severity warning)

### Quality Rules

Before submitting, ensure all checkstyle rules with severity `warning` or higher in `checkstyle.xml` pass.

## Building and Testing

### Allowed Build Commands

Only issue these Gradle commands when working as an agent.
When running as a Copilot agent (e.g. via github.com or in CI), always use `--offline` — the Gradle cache is pre-populated by the setup workflow (`.github/workflows/copilot-setup-steps.yml`). For local developer builds outside of the agent context, `--offline` is not required.

| Purpose | Command | Required |
|---|---|---|
| Compile & build | `./gradlew --offline assembleBasicDebug` | yes |
| Unit tests | `./gradlew --offline testBasicDebug` | yes |
| Checkstyle | `./gradlew --offline checkstyle` | yes |
| Instrumented tests | — only if explicitly asked — | n/a |
| PMD checks | — only if explicitly asked — | n/a |
| Any other command | — avoid — | n/a |

> **Note:** Instrumented tests and PMD checks require special preconditions (emulator, configured account, etc.) and take a long time. Do **not** run them automatically as part of verifying a code change — only run them when the user explicitly asks for it.

> **Note:** "Compile for all code" means that main, unit test, and instrumented test sources must compile. This is verified by `assembleBasicDebug` combined with `testBasicDebug`. Instrumented tests are not executed as part of this, but their compilation is covered by the build step.

### Testing Guidelines

- **Pure unit tests** go in `main/src/test/java` — prefer these whenever possible
- **Instrumented tests** go in `main/src/androidTest/java` — write them when Android framework is required, but only run them when explicitly asked by the user
- Test classes should be in the same package as the class under test
- Tests that interact with geocaching.com require a configured account on the emulator — only run these when explicitly asked by the user

### Mandatory Quality Gates


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cgeo/cgeo](https://github.com/cgeo/cgeo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
