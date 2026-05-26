---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Silo** — OSS Kotlin/Ktor replacement for the EOL Gradle Remote Build Cache Node. Drop-in HTTP
server speaking the Gradle build cache protocol. Fat-jar and multi-arch Docker. Apache-2.0.

Status: pre-scaffold. The repo currently holds docs and CI/Docker scaffolding; the Gradle build
and Kotlin source are filed as GitHub issues #1–#5 and will land via PRs.

Approved bootstrap plan: `~/.claude-work/plans/project-goal-and-archetecture-mossy-naur.md`.

## Stack

- Kotlin 2.x latest stable, JVM 21+ (virtual threads)
- Ktor 3.5, Netty engine (high-throughput streaming binary)
- Gradle with Kotlin DSL, **multi-module** layout
- SQLite WAL (`org.xerial:sqlite-jdbc`) for the metadata index
- Micrometer + Prometheus for metrics
- Logback + logstash-encoder (JSON logs)
- kotest 6 for tests (`testApplication { }` for HTTP)
- Kobweb (Compose-HTML, static export) for the admin SPA
- Shadow plugin for the fat jar; `docker buildx` for multi-arch images

**Forbidden**: Spring, Koin in core, Jackson, ORMs (Hibernate/Exposed for cache data — use plain JDBC against SQLite), `!!` (detekt blocks it), `runBlocking` outside `main` and tests.

## Gradle Build Cache Protocol

Server MUST implement the [Gradle HTTP build cache protocol](https://docs.gradle.org/current/userguide/build_cache.html#sec:build_cache_configure_remote):

- `GET /cache/{key}` → `2xx` + body bytes | `404` cache miss (NOT an error from the client's perspective)
- `PUT /cache/{key}` → any `2xx` = stored. `413` = too large (non-fatal to the Gradle client)
- `HEAD /cache/{key}` → existence check, no body
- Optional HTTP Basic auth. Anonymous-read toggle for legacy node parity.
- Honor `Expect: 100-continue` — reject (`413`/`401`) before consuming the body.
- Body is opaque binary. Stream it; never buffer.

Validate keys before touching the filesystem (regex `^[a-f0-9]{8,128}$` — opaque hex, tolerant of future Gradle hash changes). Reject anything else with `400`.

## Module layout

Multi-module Gradle build (Kotlin DSL), package root `com.chrisjenx.silo`:

```
:protocol          - CacheKey value class, content-types, errors (pure Kotlin)
:storage           - CacheStore + EvictionPolicy + Reservation interfaces, no impls
:storage-fs        - FileSystemCacheStore with 2-level sharded layout
:metadata-sqlite   - MetadataIndex impl over SQLite WAL
:metrics           - Micrometer/Prometheus wiring
:server            - Ktor app, routes, auth, config
:web               - Kobweb SPA (separate composite project under web/)
:bench             - kotlinx-benchmark/JMH
:test-fixtures     - shared kotest helpers (TmpCacheRoot, TestKeys, FakeClock)
```

The `:server` module never depends on Kobweb; the SPA is built independently and the static
export is copied into `:server/src/main/resources/static/admin/` at assembly time.

## Atomic write protocol (FS + SQLite)

The single most important invariant: **`kill -9` mid-PUT cannot corrupt the cache.**

1. Validate key → `MetadataIndex.reserve(size)` returns victim keys (async deleted).
2. Stream the body to `tmp.{key}.{uuid}` **in the same shard directory** as `final` — rename never crosses filesystems or overlay layers.
3. `fileChannel.force(true)` — data fsync.
4. `Files.move(tmp, final, ATOMIC_MOVE)` — catch `AtomicMoveNotSupportedException` and fall back to copy+delete with WARN (cross-FS root).
5. Parent dir fsync (config flag).
6. SQLite `UPSERT INTO cache_entry(...)` in a single transaction.
7. Eviction policy is notified; in-memory LRU mirror updated.

Concurrent PUTs of the same key are safe: content-addressable → identical bytes → atomic rename → last-writer-wins. **No per-key locks.** This is a deliberate decision; document if you change it.

## Storage caps & auto-cleanup

All caps enforced concurrently:

| Cap | Default | Trigger |
|---|---|---|
| `silo.storage.max-bytes` | 100 GB | LRU evict oldest by `last_access` |
| `silo.storage.max-entries` | 1,000,000 | Inode-exhaustion guard |
| `silo.storage.max-entry-bytes` | 2 GB | PUT > limit → `413` (early via `Expect: 100-continue`) |
| `silo.eviction.max-age-days` | 30 | TTL sweeper drops untouched entries |
| `silo.storage.reserved-free-bytes` | 5 GB | Stop accepting PUTs → `503` |
| `silo.storage.reserved-free-inodes` | 100,000 | Same, but for `df -i` |

**Eviction order**: TTL (age-first) → capacity-overflow (LRU) → emergency reserve. Budget-limited (`silo.eviction.max-deletes-per-cycle = 1000`) to avoid I/O storms. Background sweeper runs every 60s.

`last_access` updates are batched every 60s via a single SQLite UPDATE (cuts write volume 99%+).

## External interference handling

The server must remain consistent if another process modifies `cas/` out of band.

- Single-process lockfile: `.silo.lock` at root via `FileChannel.tryLock()`. Refuse to start if locked.
- On-read ENOENT fallback (bazel-remote pattern): SQLite says hit, disk says no → delete the row, log WARN, return 404 (looks like a cache miss to the client; self-heals).
- Periodic reconciliation sweep (default hourly): walks `cas/**` lazily; orphan blobs re-indexed, orphan rows deleted, stale `.tmp.*` files (>10 min) removed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrisjenx/silo](https://github.com/chrisjenx/silo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
