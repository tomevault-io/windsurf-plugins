---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

Java tooling requires **Java 25** (`sdk install java 25-tem`). The Gradle wrapper handles everything else.

```bash
./gradlew clean build              # Full build with tests
./gradlew clean build -x test      # Build, skip tests
./gradlew :importer:build          # Build a single module
./gradlew spotlessApply            # Apply code formatting (palantirJavaFormat, prettier, ktfmt)
./gradlew spotlessCheck            # Verify formatting
./gradlew test                     # Run all tests
./gradlew :common:test             # Tests for one module
./gradlew test --tests "*FooTest"            # Single test class
./gradlew test --tests "*FooTest.bar"        # Single test method
./gradlew :test:acceptance --info -Dcucumber.filter.tags=@acceptance   # E2E (Cucumber, hits a live network)
```

The REST module is Node.js, while Pinger and Rosetta are Go-based. While not native Java, most of the same Gradle
commands used for Java modules can be used with them:

```bash
./gradlew :pinger:clean :rest:clean :rest:monitoring:clean :rosetta:clean # Clean build artifacts
./gradlew :pinger:build :rest:build :rest:monitoring:build :rosetta:build # Full build with tests
./gradlew :pinger:test :rest:test :rest:monitoring:test :rosetta:test     # Run all tests
./gradlew :pinger:run :rest:run :rest:monitoring:run :rosetta:run         # Run the application locally
```

Run a JVM component locally via Spring Boot:

```bash
./gradlew :importer:run
./gradlew :importer:run --args='--spring.profiles.active=v2'   # Citus/sharded variant
```

Local stack (PostgreSQL plus all components):

```bash
docker compose up
```

Build/push images: `./gradlew dockerPush -PimagePlatform=linux/amd64 -PimageRegistry=... -PimageTag=...` (
see [docs/development.md](docs/development.md)).

## Architecture

A mirror node ingests record/block stream files produced by Hiero consensus nodes and serves them via several APIs. The
repo is one multi-module Gradle build (`settings.gradle.kts`). The Go and Node.js modules are integrated into the Gradle
build as well. Modules talk only via the database and Redis — no in-process cross-module calls.

### Data flow

```
Consensus nodes → S3/GCS (record + signature files) → importer → PostgreSQL → {rest, rest-java, web3, grpc, graphql, rosetta} → clients
                                                                     ↑
                                                                  monitor (synthetic load + verification)
```

The importer downloads signature files, verifies ≥ 1/3 node agreement on each record file's hash, downloads matching
record files, validates the hash chain, and ingests normalized rows.

### Modules

| Module      | Language          | Role                                                                                                                                        |
| ----------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `common`    | Java              | Shared domain entities, JPA repositories, configuration. Almost every JVM module depends on it.                                             |
| `graphql`   | Java/Spring       | GraphQL surface over the same data.                                                                                                         |
| `grpc`      | Java/Spring       | gRPC subscription endpoints (HCS topic streaming, etc.).                                                                                    |
| `importer`  | Java/Spring       | Stream-file downloader + parser + Flyway migrations. Owns the database schema (`importer/.../db/migration`).                                |
| `monitor`   | Java/Spring       | Generates synthetic transactions and validates round-trip ingestion across networks.                                                        |
| `pinger`    | Go                | Lightweight liveness/keepalive transaction submitter.                                                                                       |
| `protobuf`  | Java              | Generated protobuf bindings for the grpc module.                                                                                            |
| `rest-java` | Java/Spring       | Java replacement for newer REST endpoints (jOOQ-based queries).                                                                             |
| `rest`      | Node.js (Express) | Primary public REST API at `/api/v1/...`. Endpoint→table mapping is in [docs/rest/README.md](docs/rest/README.md).                          |
| `rosetta`   | Go                | Coinbase Rosetta API implementation.                                                                                                        |
| `test`      | Java/Cucumber     | E2E acceptance suite that drives a live network via the Hiero Java SDK. Disables the default `test` task — only the `acceptance` task runs. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hiero-ledger/hiero-mirror-node](https://github.com/hiero-ledger/hiero-mirror-node) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
