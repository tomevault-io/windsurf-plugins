---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Artemis is an interactive learning platform for programming exercises, quizzes, modeling tasks, and exams with automatic and manual assessment. It integrates with AI services (Iris for virtual tutoring, Athena for automated assessment, Hyperion for exercise creation).

## Tech Stack

- **Server**: Spring Boot 4.1 (Java 25), MySQL, Hibernate, Hazelcast
- **Client**: Angular 21, TypeScript, SCSS
- **Build**: Gradle 9.6, pnpm 11 / Node 24 (pnpm version pinned via the `packageManager` field in package.json; activate with `corepack enable`)
- **Testing**: JUnit 6, Vitest, Playwright

## Build & Development Commands

### Server

```bash
./gradlew bootRun                          # Start dev server (includes Angular build)
./gradlew bootRun -x webapp                # Server only (use with pnpm start)
./gradlew -Pprod -Pwar clean bootWar       # Production WAR (no SBOM, fast)
./gradlew -Pprod -Pwar -Psbom clean bootWar # Production WAR including server + client SBOM
```

SBOM generation (`cyclonedxBom` + `generateClientSbom`) is gated behind the `-Psbom` Gradle property. CI release-eligible jobs (pushes to `develop`/`main`/`release/*`, version tags, and published releases) set it automatically in `.github/workflows/ci-build.yml`. Local builds and PR CI ship a WAR without the SBOM — `AdminSbomResource` returns 404 and the admin UI renders an informational banner in that case.

### Client

```bash
corepack enable                      # One-time: activate the pnpm version pinned in package.json
pnpm install --frozen-lockfile       # Install dependencies (CI-style, asserts lockfile is authoritative)
pnpm install                         # Install + allow lockfile updates (for dependency changes)
pnpm start                           # Angular dev server with HMR (runs prebuild + ng serve)
pnpm run webapp:build                # Development build
pnpm run webapp:prod                 # Production build
pnpm run build                       # Alternative production build
```

### Build Output

- Client assets: `build/resources/main/static`
- Production WAR: `build/libs/Artemis-<version>.war`

### Code Quality

```bash
./gradlew spotlessCheck              # Check Java formatting
./gradlew spotlessApply              # Fix Java formatting
./gradlew checkstyleMain             # Java linting
./gradlew modernizer                 # Check for legacy API usage
pnpm run lint                        # ESLint
pnpm run lint:fix                    # Fix ESLint issues
pnpm run stylelint                   # SCSS linting
pnpm run prettier:check              # Check formatting
pnpm run prettier:write              # Fix formatting
```

### Testing

```bash
# Server (requires Docker — tests run against PostgreSQL via Testcontainers by default)
./gradlew test -x webapp                                          # All server tests (PostgreSQL)
./gradlew test --tests ExamIntegrationTest -x webapp              # Single test class
./gradlew test --tests ExamIntegrationTest.testGetExamScore       # Single test method

# Client (Vitest - preferred for new tests)
pnpm run vitest                      # Watch mode
pnpm run vitest:run                  # Single run
pnpm run vitest:coverage             # With coverage
pnpm run vitest -- path/to/spec.ts   # Single Vitest file

# E2E Tests (Playwright) — preferred way to run locally
# The script auto-kills processes on ports 8080/9000/7921, starts Postgres, server, and client.
./run-e2e-tests-local-fast.sh                              # Run all E2E tests
./run-e2e-tests-local-fast.sh --filter "Quiz"              # Run tests matching "Quiz"
./run-e2e-tests-local-fast.sh --filter "ExamAssessment|SystemHealth"  # Multiple patterns
./run-e2e-tests-local-fast.sh --stop                       # Stop all services

# Multi-node E2E (catches Hazelcast cluster / L2 cache coherence regressions)
# Boots the full production-faithful stack: Postgres, JHipster Registry (Eureka),
# ActiveMQ, 3 Artemis nodes, nginx LB, containerised Playwright. Slower than the
# single-node fast script, but the only way to reproduce multi-node bugs locally.
./run-e2e-tests-local-multinode.sh                         # Full multi-node run (build WAR + image + stack + tests)
./run-e2e-tests-local-multinode.sh --filter "Quiz"         # Multi-node, filtered
./run-e2e-tests-local-multinode.sh --skip-build --skip-up  # Quick re-run against an already-running stack
./run-e2e-tests-local-multinode.sh --stop                  # Tear everything down

# Multi-node E2E (fast variant) — same topology, host-launched JVMs instead of Docker images
# Skips the Docker image build that dominates the slow path (~5–8 min). Reuses the WAR built by
# Gradle and runs 3 java -jar processes on the host; Postgres/Eureka/ActiveMQ/nginx still run as
# containers. Use this for server-side iteration on multi-node bugs. Cold ~1–2 min, warm ~30 s.
./run-e2e-tests-local-multinode-fast.sh                       # Full run (build WAR + infra + 3 host JVMs + tests)
./run-e2e-tests-local-multinode-fast.sh --filter "Quiz"       # Filter to a subset of tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ls1intum/Artemis](https://github.com/ls1intum/Artemis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
