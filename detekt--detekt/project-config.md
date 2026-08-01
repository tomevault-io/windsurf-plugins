---
trigger: always_on
description: This document provides guidance for AI coding agents (Claude, Codex, Copilot, etc.) working on the detekt codebase. For human contributors, please see [CONTRIBUTING.md](.github/CONTRIBUTING.md).
---

# AGENTS.md - AI Coding Agent Guidelines for detekt

This document provides guidance for AI coding agents (Claude, Codex, Copilot, etc.) working on the detekt codebase. For human contributors, please see [CONTRIBUTING.md](.github/CONTRIBUTING.md).

## Project Overview

**detekt** is a static code analysis tool for Kotlin. It provides:
- Code smell analysis with 200+ built-in rules
- Highly configurable rule sets
- Multiple report formats (HTML, Markdown, SARIF, Checkstyle XML)
- Extensibility through custom rules, processors, and reports
- Gradle plugin and CLI interfaces

## Development Environment

### Prerequisites
- JDK 17+ required for the build
  - Note that some detekt-gradle-plugin tests only run on JDK 19 or lower
  - JDK 17 must be available for build-logic JVM toolchain
- Android SDK must be installed to execute certain tests. They will be skipped if the SDK is not installed. This is only an issue if making changes to detekt-gradle-plugin.

### Initial Setup
```bash
# Clone and enter the repository
git clone https://github.com/detekt/detekt.git
cd detekt

# NOTE: Only required before running Gradle plugin functional tests
# ./gradlew publishToMavenLocal

# Build (excluding slow documentation generation)
./gradlew build -x dokkaGenerate
```

### Verification Commands
```bash
# Run detekt on itself (must pass before submitting PRs)
./gradlew detektMain detektTest detektFunctionalTest detektTestFixtures

# Run all tests (including functional tests)
./gradlew test detektFunctionalTest detektFunctionalTestMinSupportedGradle

# Generate documentation (when modifying rules)
./gradlew generateDocumentation
```

## Project Structure

Key modules:
- `detekt-api/` - Public API for extending detekt
- `detekt-core/` - Core analysis engine
- `detekt-cli/` - Command-line interface
- `detekt-gradle-plugin/` - Gradle plugin (composite build in `detekt-gradle-plugin/`)
- `detekt-rules-*/` - Rule implementations by category (style, complexity, performance, etc.)
- `detekt-report-*/` - Report format implementations
- `detekt-test/` - Testing utilities

## Code Conventions

- Follow [Kotlin coding conventions](https://kotlinlang.org/docs/reference/coding-conventions.html)
- Code style is enforced by detekt via the `detekt` Gradle tasks
- Use settings from [.editorconfig](.editorconfig)
- Test classes must use the `Spec.kt` suffix
- All code in `detekt-api` and rules must be documented with KDoc

## Testing Requirements

- Use [JUnit 5](https://junit.org/junit5/docs/current/user-guide/) for testing
- For rules requiring type resolution, annotate test classes with `@KotlinCoreEnvironmentTest`
- Test new rules with `--run-rule RuleSet:RuleId` option for isolation testing

## CI/CD Pipeline

The CI workflow is located in `.github/workflows/`. Key checks:
- `./gradlew build` - Full build and test suite
- Self-analysis with detekt on all source sets
- Code coverage reporting

---

## AI Contribution Policy

This is an open-source project. All contributions, whether from humans or AI agents, are held to the same standards.

### Requirements for AI-Assisted PRs

1. **CI Checks Must Pass**
   - All automated checks must be green before merge
   - Run `./gradlew build` and `./gradlew detektMain detektTest` locally first

2. **Test Coverage**
   - Increase or maintain test coverage with your changes
   - Every new rule or feature must include comprehensive tests
   - Bug fixes should include regression tests

3. **Commit Attribution**
   - AI agents must be credited as commit co-authors
   - Use `Co-authored-by:` trailer in commit messages
   - Example:
     ```
     feat: add new rule for detecting X

     Co-authored-by: Claude <claude@anthropic.com>
     Co-authored-by: Your Name <you@example.com>
     ```

4. **No Batch AI Responses**
   - Do not use AI to reply to review comments in bulk
   - Each response should be contextual and thoughtful
   - Human judgment must guide the conversation

5. **Trust But Verify**
   - AI-generated code must be reviewed by humans
   - Verify that generated code is accurate and idiomatic
   - Ensure alignment with detekt's architecture and conventions

6. **No Spam PRs**
   - Do not create PRs to artificially inflate contribution counts
   - Each PR must provide genuine value to the project
   - Trivial changes should be bundled appropriately

7. **Review Policy**
   - PRs should not be reviewed by the same AI model that authored them
   - Human reviews are always welcome and encouraged
   - Reviews from different AI models (not the author) are acceptable

### What We Expect from AI-Generated Contributions

- **Accuracy**: Code must be correct and well-tested
- **Idiomatic**: Follow Kotlin best practices and detekt conventions
- **Documented**: Include proper KDoc for public APIs and rules
- **Focused**: PRs should address a single concern
- **Transparent**: Clearly indicate AI involvement in the PR description

### Unacceptable AI Usage

- Submitting PRs with no human review or oversight
- Using AI to generate meaningless or low-quality contributions
- Copying AI responses directly into review comments without context

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [detekt/detekt](https://github.com/detekt/detekt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
