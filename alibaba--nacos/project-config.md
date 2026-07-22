---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, Cursor, GitHub Copilot, etc.) when working with the Nacos repository. For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md).
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, GitHub Copilot, etc.) when working with the Nacos repository. For human contributors, see [CONTRIBUTING.md](./CONTRIBUTING.md).

## AI Contribution Guidelines

- **Do NOT post AI-generated comments** on issues or PRs. Discussions are for humans only.
- **Discuss before implementing**: Ensure the implementation direction is agreed upon with maintainers in the issue comments before starting work.
- **Spec-first coding is mandatory**: Before changing behavior, APIs, SDKs,
  plugins, storage, runtime flow, or domain semantics, AI agents MUST read the
  relevant specs under [`specs/`](./specs/README.md) and treat them as the rule
  source for the implementation.
- **Discuss spec-impacting changes before coding**: If a change touches behavior
  covered by an existing spec, or exposes a gap between the code and the spec,
  AI agents MUST discuss the implementation direction and the required spec
  update with maintainers before starting the code change.
- **Update specs with the design**: Any design proposal that changes or
  clarifies spec-covered behavior MUST include the corresponding spec updates
  in the same change set. When the design is large or controversial, prefer a
  spec/design-only PR first, then follow with implementation PRs.
- **API IT impact comes first**: Before adding, changing, deleting, or
  deprecating any HTTP API, AI agents MUST analyze the affected
  `test/openapi-test` coverage and update the API IT scenario matrix, test
  cases, and coverage registry in the same change set. If the functional path
  cannot be exercised in standalone IT, cover boundary/error scenarios and
  document the reason.
- **Java SDK IT impact comes first**: Before adding, changing, deleting, or
  deprecating any public Java SDK interface, factory, model, listener behavior,
  lifecycle behavior, or exception mapping, AI agents MUST analyze and update
  `test/java-sdk-test` coverage, including scenario documentation. If the
  end-to-end success path is impractical, cover SDK parameter validation,
  boundary behavior, and controlled exceptions.
- **Disclose AI usage**: When a significant part of a commit is AI-generated, add a trailer to your commit message:
  ```
  Assisted-by: Claude Code
  ```
- **Follow [CONTRIBUTING.md](./CONTRIBUTING.md)** for all contribution processes.

## Repository Overview

Nacos (Dynamic Naming and Configuration Service) is an easy-to-use platform designed for dynamic service discovery, configuration management, and AI agent management. It helps you build cloud-native applications and AI Agent applications easily. Key capabilities: service discovery, dynamic configuration, dynamic DNS service, service/metadata management, and AI registry (Prompt, MCP, A2A).

**Current Version**: 3.2.1-SNAPSHOT | **Main Branch**: `develop` | **Java**: JDK 17+ (client modules: JDK 8+) | **Build**: Maven 3.2.5+

## Core Architecture

Key modules and their roles:

- **api / client / client-basic**: Client-facing APIs, gRPC definitions, SDK (Java 8 compatible)
- **common**: Shared utilities, HTTP client, notify center, executor
- **config**: Configuration management server
- **naming**: Service discovery and registration server
- **core**: Core server infrastructure (cluster, distributed consensus)
- **consistency**: JRaft-based CP protocol + custom Distro AP protocol
- **auth**: Authentication and authorization
- **plugin / plugin-default-impl**: Extensible plugin system (Java SPI). Types: auth, visibility, datasource dialect, config change, encryption, trace, environment, control, AI pipeline, AI storage
- **console / console-ui**: Web UI backend (Spring Boot) and frontend (React)
- **ai / copilot / ai-registry-adaptor**: AI Agent support, Copilot integration, and AI registry adaptor
- **sys**: System environment utilities and JVM parameter management
- **bootstrap / server**: Server startup and aggregation
- **persistence**: Data persistence with multi-database support (Derby, MySQL, PostgreSQL)
- **maintainer-client**: Internal maintenance client
- **lock**: Distributed lock support

Communication: **gRPC** (primary) + **HTTP/REST** (legacy compatibility). Protobuf definitions in `api/src/main/proto/`.

## Build & Test Commands

```bash
# Full build (skip tests)
mvn '-Prelease-nacos,!dev' -Dmaven.test.skip=true clean install -U

# Run all unit tests
mvn test

# Run standalone-server integration tests
mvn -pl test/openapi-test -Pintegration-test -DskipTests=false verify
mvn -pl test/java-sdk-test -Pjava-sdk-integration-test -DskipTests=false verify
mvn -pl test/maintainer-sdk-test -Pmaintainer-sdk-integration-test -DskipTests=false verify

# Format code (run before commit)
mvn spotless:apply

# Pre-submission checks (MUST pass before PR)
mvn -B clean compile apache-rat:check checkstyle:check spotbugs:check spotless:check -DskipTests
```

### Mandatory Formatting Before Commit

Before committing Java code or tests, AI agents MUST run Spotless for the
affected module or nearest aggregator:

1. Run `mvn spotless:apply` first.
2. Run `mvn spotless:check` for the same scope.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alibaba/nacos](https://github.com/alibaba/nacos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
