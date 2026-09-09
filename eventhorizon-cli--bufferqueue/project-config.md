---
trigger: always_on
description: You are an AI coding assistant for this repository.
---

# AGENTS.md

You are an AI coding assistant for this repository.

## Scope and working style

- These instructions apply repository-wide unless a more specific `AGENTS.md` exists below the target path.
- Follow [`.editorconfig`](.editorconfig) and nearby code before introducing a new style or abstraction.
- Keep changes focused, maintainable, and production-ready. Preserve unrelated user changes.
- The projects define the supported target frameworks and language version. Do not change either policy incidentally.
- The library, sample, and test projects target `net8.0` and `net10.0`. Benchmarks intentionally target `net10.0`
  only.
- This repository uses the standard MIT License in `LICENSE`. Do not add file-level license headers.

## References

Use the design-note index as the source of truth for detailed, evolving guidance instead of duplicating it here:

- [Design-note index](docs/README.md): English architecture, consumer, routing, and storage notes.
- [Architecture](docs/design/architecture.md): package boundary, queue ownership, dependency injection, and extension
  points.
- [Consumer model](docs/design/consumer-model.md): consumer groups, pull and push consumers, wake-up behavior, and
  delivery semantics.
- [Partitioning and concurrency](docs/design/partitioning-and-concurrency.md): routing strategies, selector
  requirements, locking, and visibility rules.
- [Memory storage](docs/design/memory.md): segment layout, bounded capacity, recycling, and offsets.
- [Memory-mapped-file storage](docs/design/memory-mapped-file.md): record format, persistence, recovery, retention,
  serializers, and flush boundaries.
- The matching `*.zh-CN.md` note is the Simplified Chinese counterpart of each English design note.

Read the relevant note before changing an architectural boundary, storage behavior, partitioning rule, delivery
semantic, persistence format, or consumer lifecycle.

## Architecture constraints

- `IBufferQueue` is the public entry point. It resolves typed topic queues registered as keyed `IBufferQueue<T>`
  services and exposes producers for runtime-selected topics.
- `BufferQueue<TItem>` contains shared typed queue behavior. It owns consumer-option validation, consumer-group
  registration, partition assignment, pull-consumer creation, and the topic producer.
- `BufferPullConsumer<TItem>` is the common pull-consumer implementation. Do not create storage-specific copies of
  group assignment, batch consumption, commits, or waiting behavior.
- Storage-specific behavior belongs behind `IBufferPartition<TItem>`. Keep upper-level queue behavior out of storage
  implementations unless it is truly storage-specific.
- Memory storage belongs in `src/BufferQueue/Memory/`. MemoryMappedFile storage belongs in
  `src/BufferQueue.MemoryMappedFile/`.
- `BufferQueue.MemoryMappedFile` is an optional assembly that depends on `BufferQueue` and uses shared internal queue
  abstractions through friend-assembly access. The core `BufferQueue` project must not reference the MMF project.
  Preserve the current public namespaces and `.UseMemoryMappedFile(...)` registration API.
- A new storage implementation should supply its partitions, producer, queue type, options, and DI registration while
  reusing the common queue and consumer layers.
- Use keyed DI registrations for typed topics. Do not add a factory when keyed services already express the ownership
  and lifetime correctly.

## Storage and persistence constraints

### Memory

- Memory mode stores items in linked, fixed-size segments and can enforce a total bounded capacity through
  `MemoryBufferQueueOptions.BoundedCapacity`.
- A memory segment may be recycled only after every consumer group has advanced past it. A slow group must never lose
  unread data.
- `MemoryBufferQueueOptions.FullMode` defaults to `Wait`: `ProduceAsync` asynchronously waits with cancellation support
  when a bounded queue has no capacity. `Fail` throws `BufferQueueFullException` immediately. `TryProduceAsync` never
  waits for bounded capacity and returns `false` when admission is unavailable.
- Batch admission is all-or-nothing. In `Wait` mode, `ProduceAsync` waits for the complete batch, and a batch larger
  than the configured capacity is invalid. Capacity is released when the minimum committed position across all known
  consumer groups advances, including partial segments.

### MemoryMappedFile

- MMF records are `int32 little-endian payload length`, payload bytes, and one record-end marker byte. A segment-end
  marker is an `int32` length of `-1`.
- Go through `OffsetCheckpoint` for all `producer.offset`, `earliest.offset`, and consumer-offset reads and writes.
  Do not duplicate checkpoint file IO in partition code.
- When retention is enabled, reclaim only complete segments below the minimum committed offset of every known consumer
  group. Advance `earliest.offset` before disposing mappings and deleting old segment files.
- Recovery and reads must never recreate a missing segment inside the retained range. Invalid checkpoints, offsets,
  record boundaries, or retained files must fail fast with a clear exception instead of silently resetting progress.
- Group directories should remain readable whenever possible. Escape only characters that are unsafe in one path
  component.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eventhorizon-cli/BufferQueue](https://github.com/eventhorizon-cli/BufferQueue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
