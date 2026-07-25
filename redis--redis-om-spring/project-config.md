---
trigger: always_on
description: **Last updated**: 2026-05-29
---

# Redis OM Spring Agent Guide

**Last updated**: 2026-05-29

## Project Overview

Redis OM Spring is a Gradle multi-module Java library that extends Spring Data Redis
with higher-level Redis-native modeling and query features. It is published to Maven
Central and consumed as a dependency by Spring Boot applications — not a deployable
service.

Agents work from the approved spec for the active feature. If a feature has no
approved spec yet, work from this file and existing conventions only.

## What the Library Does

Major capabilities:

- `@Document` for Redis JSON persistence
- Enhanced `@RedisHash` with Redis Query Engine integration
- `RedisDocumentRepository` and `RedisEnhancedRepository`
- Declarative indexing: `@Indexed`, `@Searchable`, `@GeoIndexed`, `@VectorIndexed`
- `EntityStream` — fluent, type-safe querying and aggregations
- Query derivation, `@Query`, and `@Aggregation`
- ULID generation for string IDs
- Multi-tenant dynamic index naming via SpEL
- Index migration: blue-green, dual-write, and in-place strategies
- Probabilistic data structures: Bloom, Cuckoo, Count-Min
- AI-powered vectorization and embedding via the `redis-om-spring-ai` module

## How Customers Use It

1. Run Redis 8+ or Redis Stack (JSON + Query Engine required).
2. Add `redis-om-spring` (and optionally `redis-om-spring-ai`) as a dependency.
3. Annotate domain classes with `@Document` or enhanced hash annotations.
4. Enable repositories with `@EnableRedisDocumentRepositories` or `@EnableRedisEnhancedRepositories`.
5. Use Spring repository interfaces, query derivation, `@Query`/`@Aggregation`, or `EntityStream`.

Redis defaults to `localhost:6379`. A `docker-compose.yml` at the repo root starts Redis Stack for local work.

## Supported Redis Targets

The library requires the Query Engine and JSON capabilities. This means:

- Redis 8+ (with bundled modules)
- Redis Stack
- Redis Enterprise
- Redis Cloud

Plain Redis OSS without modules does not support the full feature set.

## Source of Truth

Read in this order:

1. The active feature spec in `specs/<<slug>>/`
2. `README.md` — user-facing project overview and public API examples
3. `docs/content/modules/ROOT/pages/` — Antora documentation site source
4. This file

## Before Coding

1. Read the active feature spec, plan, and tasks.
2. Read `README.md` for the public API surface and user expectations.

Do not implement behavior changes without an approved spec.

## Module Map

| Module | Purpose |
|---|---|
| `redis-om-spring/` | Core library — annotations, repositories, indexing, search |
| `redis-om-spring-ai/` | Optional AI/vectorization extension (Spring AI + DJL) |
| `tests/` | **All integration and unit tests go here** — not inside the library modules |
| `docs/` | Antora documentation site + Javadoc generation |
| `demos/` | Sample Spring Boot applications |
| `specs/` | Feature specs (one folder per feature) |

Key config files: `gradle.properties`, `build.gradle`, `gradle/build-conventions.gradle`, `settings.gradle`.

## Best Demos to Run First

```bash
./gradlew :demos:roms-documents:bootRun      # JSON document mapping + REST CRUD
./gradlew :demos:roms-multitenant:bootRun    # Dynamic index naming, tenant isolation
./gradlew :demos:roms-hybrid:bootRun         # Hybrid text + vector search
./gradlew :demos:roms-vectorizers:bootRun    # Automatic embedding + semantic search
```

## New Feature Gate

Every new feature requires three approved artifacts before any implementation code is written:

1. `specs/<slug>/spec.md` — approved requirements and acceptance scenarios
2. `specs/<slug>/plan.md` — approved implementation design
3. `specs/<slug>/tasks.md` — approved execution checklist

**Do not write implementation code until all three are approved.**

Branch names must be **under 50 characters**: a short type prefix, a `/`, then the slug.

```
feat/730-sentinel-connection        ✓
fix/RED-1234-sentinel-connection    ✓
docs/update-vector-search-guide     ✓
chore/upgrade-jedis                 ✓
feat/add-a-very-long-description    ✗  too long
```

Common prefixes: `feat/`, `fix/`, `docs/`, `chore/`, `refactor/`, `test/`.

## Working Rules

- Stay within the approved spec.
- Keep changes focused and reviewable.
- Do not refactor unrelated code.
- Add or update tests when behavior changes.
- Tests go in `tests/src/test/java/com/redis/om/spring/` — never inside `redis-om-spring/` or `redis-om-spring-ai/`.
- Do not add dependencies without justification.
- Do not hard-code environment-specific values.
- Do not weaken, skip, or delete tests to force a change through.
- Update docs under `docs/content/modules/ROOT/pages/` when changing user-visible behavior.
- Search existing specs before introducing a new pattern.
- Do not invent folder layouts or architecture that are not in source-of-truth docs.
- **This repo has no Maven wrapper.** Use `./gradlew` only. Do not reference `mvnw`.

## Known Inconsistencies to Avoid Repeating

- Some older demo and docs files reference `mvnw` commands — ignore these, the repo is Gradle-only.
- The Java minimum requirement varies across docs: the build toolchain requires Java 21; some README text says Java 17. Treat **Java 21** as authoritative.

## When To Ask

Ask for clarification when:

- the spec is ambiguous on a critical point

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redis/redis-om-spring](https://github.com/redis/redis-om-spring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
