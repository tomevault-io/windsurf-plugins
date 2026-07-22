---
trigger: always_on
description: This file provides guidance to AI Agents (Claude Code, Codex, Gemini, Cursor etc.) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to AI Agents (Claude Code, Codex, Gemini, Cursor etc.) when working with code in this repository.

## Project Overview

Axon Framework is a framework for building evolutionary, event-driven microservice systems based on Domain-Driven Design (DDD), Command-Query Responsibility Separation (CQRS), and Event Sourcing principles. This is **Axon Framework 5**, a major version under development with significant architectural changes from version 4.

### Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

### Workflow Orchestration

1.  **Spec-first**: Enter plan mode for non-trivial tasks (3+ steps or architectural decisions).
2.  **Subagent strategy**: Use subagents liberally to keep main context clean. Offload research and parallel analysis. One task per subagent.
3.  **Self-improvement**: After corrections, update the relevant CLAUDE.md or a file under `.claude/rules/`, or introduce a new one. Write rules that prevent the same mistake.
4.  **Verification**: Run tests, check build, suggest user verification. Ask: "Would a staff engineer approve this?"
5.  **Elegance**: For non-trivial changes, pause and ask "is there a more elegant way?" Skip for simple fixes.
6.  **Autonomous bug fixing**: When given a bug report, just fix it. Point at logs/errors, then resolve. Zero hand-holding.

### Key Architectural Principles

- **JDK 21 Base**: Framework requires Java 21. During implementation use Java 21 features like sealed classes.
- **Reactive Support**: Framework supports both reactive and imperative programming styles without enforcing either
- **Modular Design**: Breaking apart monolithic modules into focused, composable components
- **No ThreadLocals**: Internally avoids ThreadLocals, only used at edges for imperative style
- **Composition over Inheritance**: Favors composition patterns throughout the codebase
- **Declarative Configuration**: Moving from annotation-heavy to declarative configuration approaches

## Branching Strategy

| Branch Pattern | Purpose |
|---|---|
| `main` | Next minor or major release. Always contains all commits — patch branch changes flow upward into `main`. |
| `axon-[2-5].[0-12].x` | Patch release branches (e.g., `axon-5.0.x`). Changes to a patch branch are merged upward through higher patch branches and ultimately into `main`. |
| `bug/[issue-number]/[name]` | Bug fixes. Prefix groups all bug branches together; issue number sub-groups related branches. |
| `enhancement/[issue-number]/[name]` | Enhancements to existing features. |
| `feature/[issue-number]/[name]` | New features. |
| `documentation/[issue-number]/[name]` | Documentation-only changes. |
| `dependency-upgrade/[issue-number]/[name]` | Dependency version upgrades. |

## Commit Guidelines
* use a subject line
* separate the subject line from the description by an empty line
* focus on the how and why in the description, not the what
* use the actual issue number at the start of the subject line, for example `[#2343]`
* reference any related issue numbers in the commit message
* avoid including unrelated changes in the same commit

## Build Commands

Maven wrapper is used (`./mvnw`). Key commands:

```bash
# Full build with tests
./mvnw clean verify

# Build skipping tests
./mvnw clean install -DskipTests

# Build with code coverage (JaCoCo)
./mvnw -Dcoverage clean verify

# Build including example modules
./mvnw -Pexamples clean verify

# Build with integration tests
./mvnw -Pintegration-test clean verify

# Run all unit tests
./mvnw clean test

# Run a single test class
./mvnw test -pl messaging -Dtest=SimpleCommandBusTest

# Run a single test method
./mvnw test -pl messaging -Dtest=SimpleCommandBusTest#methodName

# Run integration tests for a specific module
./mvnw -Pintegration-test verify -pl integrationtests -Dtest=ClassName
```

## Test Guidelines
- When writing tests avoid Mock whenever possible, try to use simplest implementation. Or use some recording implementations if needed. Ask me if it's available or you need to create your own, if you cannot find. Always ask me before doing into Mocking or custom implementation if you don't find existing one.
- **Mockito Spy is justified** when behavior-based assertions alone cannot validate the interaction — e.g., verifying a decorator **prevents** calls to the delegate (`verify(delegate, never())`), or verifying the delegate receives **correct arguments** (`verify(delegate).method(eq(expected))`). Wrap a real implementation with `spy()` and use `verify`/`clearInvocations` as needed. Outside of these cases, prefer state-based assertions.
- **While testing do not focus on implementation details like implemented interfaces, always test the behaviour by API usage.**
- **Test Utilities**: Located in `messaging/src/test/java/org/axonframework/utils/`
- **Additional Rules**:
    - Always use JUnit5, AssertJ (prefer AssertJ assertions over normal JUnit5), Awaitility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AxonIQ/AxonFramework](https://github.com/AxonIQ/AxonFramework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
