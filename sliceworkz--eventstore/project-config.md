---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Java-based EventStore library implementing the Dynamic Consistency Boundary (DCB) specification. The codebase is organized as a Maven multi-module project providing event storage abstractions with multiple backend implementations.

**Core Modules:**
- `sliceworkz-eventstore-api`: Core API interfaces and contracts
- `sliceworkz-eventstore-impl`: Implementation of the EventStore
- `sliceworkz-eventstore-infra-inmem`: In-memory storage backend (for development/testing)
- `sliceworkz-eventstore-infra-postgres`: PostgreSQL storage backend (production-ready)
- `sliceworkz-eventstore-tests`: Shared test scenarios
- `sliceworkz-eventstore-examples`: Example usage code

## Build Commands

**Build entire project:**
```bash
mvn clean install
```

**Build specific module:**
```bash
cd sliceworkz-eventstore-infra-postgres
mvn clean install
```

**Run tests:**
```bash
mvn test
```

**Run specific test:**
```bash
mvn test -Dtest=EventStoreBasicTest
```

**Skip tests during build:**
```bash
mvn clean install -DskipTests
```

## Architecture

### Core Concepts

**EventStore:**
- Main entry point for interacting with the event storage system
- Obtained via `EventStoreFactory.get().eventStore(eventStorage)`
- Provides access to event streams via `getEventStream()`

**EventStream:**
- Identified by `EventStreamId` which consists of a context and optional purpose
- Supports both reading (via `query()`) and writing (via `append()`)
- Type-safe through generic parameter `<DOMAIN_EVENT_TYPE>`
- Combines `EventSource` (reading) and `EventSink` (writing) interfaces

**Event:**
- Record type containing: `stream`, `type`, `reference`, `data`, `tags`, `timestamp`
- Data is the actual domain event (typically a sealed interface with record implementations)
- Tags enable dynamic querying and consistency boundaries
- Created via `Event.of(data, tags)` for ephemeral events or full constructor for persisted events

**EphemeralEvent:**
- Lightweight event representation before persistence (no stream, reference, or timestamp)
- Converted to full `Event` upon appending to a stream
- Created via `Event.of(data, tags)`

**Tags:**
- Key-value pairs attached to events for dynamic retrieval
- Enable querying events across different event types
- Core to the Dynamic Consistency Boundary pattern
- Created via `Tags.of("key", "value")` or `Tags.of(Tag.of("key", "value"))`

**EventFilter:**
- Pure matching criteria: event types, tags, and an optional "until" temporal boundary
- Does not carry traversal semantics (direction, limit) — those belong to `EventQuery`
- Can match all (`EventFilter.matchAll()`), none (`EventFilter.matchNone()`), or specific criteria
- Created via `EventFilter.forEvents(eventTypesFilter, tags)`
- Used by `AppendCriteria` for optimistic locking (where direction/limit are irrelevant)

**EventQuery:**
- Wraps an `EventFilter` together with traversal semantics (direction and limit)
- Use `EventQuery.filter()` to extract the pure matching criteria
- Can match all (`EventQuery.matchAll()`), none (`EventQuery.matchNone()`), or specific criteria
- Supports backward direction (`.backwards()`) and result limits (`.limit(n)`)
- Created via `EventQuery.forEvents(eventTypesFilter, tags)`

**AppendCriteria:**
- Controls optimistic locking when appending events
- Contains an `EventFilter` and an optional `EventReference` for the expected last event
- If new matching events are found after the reference, append fails with `OptimisticLockingException`
- Use `AppendCriteria.none()` for simple appends without locking
- Use `AppendCriteria.of(eventQuery, reference)` or `AppendCriteria.of(eventFilter, reference)` for conditional appends

**Projection:**
- Combines an `EventQuery` with an `EventHandler`
- Processes all events matching the query criteria
- Used for building read models from event streams
- Optionally defines an `initQuery()` for the savepoint pattern (see below)

**Projection initQuery (Savepoint Pattern):**
- Projections can define an optional `initQuery()` (default returns `null`) that runs before the main `eventQuery()`
- Enables the savepoint pattern: a backward query with limit 1 finds the most recent savepoint event that summarizes prior state
- The `Projector` executes `initQuery()` first, passes results to `when()`, then uses the last event's reference as the cursor for `eventQuery()`
- Savepoint events are pure domain events — no special framework support needed
- When no savepoint exists, the main `eventQuery()` replays from the beginning (graceful degradation)
- When bookmarking is enabled on the `Projector`, `initQuery()` is ignored (a warning is logged at build time)
- The `initQuery()` and `eventQuery()` should query different event types to avoid double-processing and to allow recovery from buggy savepoints

### Storage Implementations

**In-Memory (Development/Testing):**
```java
EventStorage storage = InMemoryEventStorage.newBuilder().build();
EventStore store = EventStoreFactory.get().eventStore(storage);

// Or use convenience method to get EventStore directly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sliceworkz/eventstore](https://github.com/sliceworkz/eventstore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
