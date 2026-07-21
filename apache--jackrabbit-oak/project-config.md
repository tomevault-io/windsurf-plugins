---
trigger: always_on
description: Apache Jackrabbit Oak is a scalable, high-performance hierarchical content repository
---

# AGENTS.md - AI Agent Instructions for Apache Jackrabbit Oak

## Project Overview

Apache Jackrabbit Oak is a scalable, high-performance hierarchical content repository
implementing the JCR (Java Content Repository) specification. It is a multi-module Maven
project with ~47 modules written in Java 11.

## General Guidelines

- When working on a specific area, first read the area-specific documentation before
  making changes (see doc links in the module tables below)
- New code must have >80% test coverage. Security modules (`oak-security-spi`,
  `oak-auth-*`, `oak-authorization-*`) require 100% test coverage
- Do not weaken or remove existing test assertions to make the build pass. If a test
  fails after a change, investigate the root cause. Only modify existing tests when the
  behavioral change is intentional and explicitly requested
- Write self-descriptive, easy-to-read code. Prefer minimal changes — AI tends to add
  more lines than necessary; aim to be concise
- Avoid trivial comments. Only add comments where the logic is complex and not
  obvious from the code itself
- Use feature toggles for non-trivial changes. If the change is a bug fix, the toggle
  should be enabled by default. If the change introduces a new feature, the toggle should
  be disabled by default

## Build Commands

```bash
# Full build (skip tests for speed)
mvn clean install -DskipTests

# Fast build (no tests, no coverage)
mvn clean install -Pfast

# Build a single module (use -DskipTests only for modules you did not change)
mvn clean install -pl oak-core -DskipTests

# Build a module and its dependencies (use -DskipTests only for unchanged dependencies)
mvn clean install -pl oak-core -am -DskipTests

# Rebuild a module and all modules that depend on it (use after SPI/API changes)
mvn clean install -pl oak-store-spi -amd -DskipTests

# Full build with tests
mvn clean install

# Build with integration tests
mvn clean install -PintegrationTesting
```

## Test Commands

```bash
# Run tests for a single module
mvn test -pl oak-core

# Run a specific test class
mvn test -pl oak-core -Dtest=TreeTest

# Run a specific test method
mvn test -pl oak-core -Dtest=TreeTest#testMethodName

# Run integration tests for a module
mvn verify -pl oak-store-document -PintegrationTesting

# Run with code coverage
mvn verify -pl oak-core -Pcoverage -Dskip.coverage=false
```

Always run tests for the modules you modified. Use `-DskipTests` only when building
dependencies that you did not change.

**Test framework:** JUnit 4 (4.13.1) with Mockito 5.x (loaded as Java agent).
Some modules also use EasyMock. Do not introduce JUnit 5 unless explicitly requested.

**Test fixtures:** Tests may run against multiple backend fixtures: `SEGMENT_TAR` (default),
`DOCUMENT_NS` (MongoDB), `DOCUMENT_RDB`, `SEGMENT_AWS`, `SEGMENT_AZURE`.

## Repository Structure

**Note:** Individual submodules may have their own `AGENTS.md` file with module-specific
instructions. When working within a particular module, check for and read its `AGENTS.md`
before making changes.

All modules grouped by area. The **Description** column provides additional context
for understanding each module's role.

### API & Core

| Module | Purpose | Description                                                                                                                                                                                            |
|--------|---------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `oak-api` | Public Oak API | Defines `ContentRepository`, `ContentSession`, `Root`, `Tree` and other core interfaces for accessing repository content. See: `oak-api/README.md`, `oak-doc/src/site/markdown/oak_api/error_codes.md` |
| `oak-core` | Core repository implementation | MVCC-based transactional content model, node state management, commit handling, and internal kernel. See: `oak-doc/src/site/markdown/archigreat l-model.md`                                            |
| `oak-core-spi` | Core SPI interfaces | Service Provider Interfaces for plugins and storage backends (NodeStore, Commit, etc.)                                                                                                                 |
| `oak-commons` | Shared utilities | Common utility classes used across all Oak modules                                                                                                                                                     |
| `oak-jackrabbit-api` | Jackrabbit API extensions | Additional API beyond the JCR spec (e.g., extended session, security). See: `oak-jackrabbit-api/README.md`                                                                                             |
| `oak-jcr` | JCR API binding | Implementation of the javax.jcr (JCR 2.0) specification on top of Oak's content model                                                                                                                  |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/jackrabbit-oak](https://github.com/apache/jackrabbit-oak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
