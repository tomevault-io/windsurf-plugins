---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sourcerer is an opinionated, functional, and storage-agnostic framework for implementing CQRS (Command Query Responsibility Segregation) with event sourcing in Java 8. It was developed by Elder (elder.org) and is designed to support functional-style programming with immutable data structures.

**Status**: effectively frozen. Elder's remaining Spring Boot 1.5 and Jooby services consume sourcerer unchanged; new services integrate with the KurrentDB client directly rather than through this framework. Backwards compatibility is the top priority for any change here.

## Build System and Commands

Gradle multi-module project (wrapper: Gradle 7.4.2). Use `./gradlew` for all builds.

```bash
# What CI runs (in this order — see .circleci/config.yml)
./gradlew checkstyleMain checkstyleTest test
./gradlew integrationTest

# Build all modules
./gradlew build

# Single module / single test
./gradlew :sourcerer-core:test
./gradlew :sourcerer-core:test --tests "*.DefaultCommandTest"
./gradlew :sourcerer-esjc-tests:integrationTest
```

Checkstyle failures break the build (`ignoreFailures = false`); config at `infra/codehealth/checkstyle.xml`.

### Integration Tests

Integration tests live in dedicated modules that contain only an `integrationTest` source set: `sourcerer-esjc-tests` and `sourcerer-eventstore-grpc-tests`. Both extend shared Kotlin test bases from `sourcerer-eventstore-test-utils` (`EventRepositoryIntegrationTestBase`, `EventStreamsIntegrationTestBase`, `EventSubscriptionIntegrationTestBase`), so the two storage backends run the same test suite.

They expect an EventStore instance on localhost — esjc connects via TCP on port 1113, grpc on port 2113. Start one with:

```bash
docker compose -f infra/eventstore/docker-compose.yaml up -d
```

(EventStore 21.10.2, insecure mode, external TCP enabled — same configuration CI uses.)

## Architecture

### Core Concepts (sourcerer-core)

The framework is built around these fundamental abstractions:

**EventRepository** (`org.elder.sourcerer.EventRepository`)
- Storage-agnostic interface for persisting and reading events
- Supports reading from individual streams or all events
- Provides reactive Publishers for event subscriptions
- Implementations must handle versioning and optimistic concurrency

**AggregateRepository** (`org.elder.sourcerer.AggregateRepository`)
- Higher-level abstraction built on EventRepository
- Understands aggregate state reconstruction from events
- Manages aggregate lifecycle (load, save, append)

**Command** (`org.elder.sourcerer.Command`)
- Represents an operation to update an aggregate
- Can be configured with: aggregate ID, expected version, parameters, atomicity requirements
- Commands are created from Operations via CommandFactory
- Supports metadata decoration for cross-cutting concerns (auth, timestamps, etc.)

**Operation** (`org.elder.sourcerer.Operation`)
- Pure business logic: (state, params) → events
- Does NOT directly read/persist events (allows dry-run and easy testing)
- Created from method references using `Operations` utility class
- Supports various handler types in `org.elder.sourcerer.functions` package

**AggregateProjection** (`org.elder.sourcerer.AggregateProjection`)
- Defines how to rebuild aggregate state from event sequence
- Used by commands to preview state changes before commit
- Functional interface: events → state

**Subscription** (`org.elder.sourcerer.EventSubscription`)
- Consume and react to events as they occur
- Self-manage position tracking in event stream
- Must be idempotent (events may replay on reconnect/restart)
- Created via `EventSubscriptionFactory` with `EventSubscriptionHandler`
- Supports automatic restart with exponential backoff and dynamic batching

### Module Structure

**sourcerer-core**
- Core abstractions and interfaces; no storage backend dependencies
- Functional handler interfaces in `org.elder.sourcerer.functions`
- Reactive Streams support via `org.reactivestreams`

**sourcerer-eventstore-grpc**
- EventStore implementation using the official gRPC client (`com.eventstore:db-client-java` 3.0.2)
- The implementation Elder services actually consume

**sourcerer-esjc**
- EventStore implementation using esjc (Java 8 native TCP client, no Akka)
- The esjc client library is unmaintained and Elder services no longer use this implementation — the README's "esjc preferred" note is stale
- esjc version set by `escjVersion` in `gradle.properties` (property name typo is real — don't "fix" it)

**sourcerer-kotlin**
- Kotlin support utilities (`EventStreams`, snapshotting via `SnapshottingSupport`/`SnapshotEvents`)
- Depends on sourcerer-esjc; helpers for Kotlin sealed-class events

**sourcerer-crud**
- CRUD operation utilities and helpers for common command patterns

**sourcerer-eventstore-test-utils**
- Shared integration test bases (Kotlin) used by both `-tests` modules

**sourcerer-esjc-tests / sourcerer-eventstore-grpc-tests**
- Integration tests only (no main source)

## Key Design Principles

1. **Functional First**: Prefer immutable objects and pure functions. State is a function over events.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elder-oss/sourcerer](https://github.com/elder-oss/sourcerer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
