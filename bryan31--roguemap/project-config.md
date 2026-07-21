---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Build and Test Commands

```bash
# Compile the project (all modules)
mvn clean compile

# Run all tests (all modules)
mvn test

# Run tests for a specific module
mvn test -pl roguemap-core
mvn test -pl roguemap-memory
mvn test -pl roguemap-memory-pro
mvn test -pl roguemap-embedding

# Run a specific test class
mvn test -Dtest=MapFunctionalTest

# Run multiple test classes
mvn test -Dtest=LinkedQueueFreeListTest,QueueCrashRecoveryTest

# Run tests by pattern
mvn test -Dtest=*ComparisonTest

# Release build (GPG signing + publish to Maven Central)
mvn clean deploy -P release
```

## Module Structure

This is a multi-module Maven project:

| Module | Java | Description |
|---|---|---|
| `roguemap-core` | 8+ | Core off-heap storage library (zero mandatory deps) |
| `roguemap-embedding` | 8+ | Universal `EmbeddingProvider` implementation; zero extra deps |
| `roguemap-memory` | 8+ | AI memory layer; HNSW vector index via `jelmerk/hnswlib-core` |
| `roguemap-memory-pro` | 11+ | AI memory layer; higher-performance HNSW via `datastax/jvector` |

`roguemap-memory` and `roguemap-memory-pro` are structurally identical except for the vector index backend (`HnswVectorIndex` vs `JVectorIndex`). Both depend on `roguemap-core`.

`roguemap-embedding` provides `UniversalEmbeddingProvider` — a single class that works with any OpenAI `/v1/embeddings`-compatible service (OpenAI, Mistral, Jina, Voyage, Ollama in OpenAI-compat mode, Alibaba DashScope, Zhipu GLM, etc.) using only `HttpURLConnection`. The older `OpenAIEmbeddingProvider` and `OllamaEmbeddingProvider` in the memory modules are `@deprecated` in favor of this class.

## Architecture Overview

RogueMap is a high-performance embedded storage library using memory-mapped files for off-heap storage. Java 8+, zero mandatory dependencies. Provides four data structures: RogueMap (key-value store), RogueList (doubly-linked list), RogueSet (concurrent set), and RogueQueue (FIFO queue with linked/circular modes).

### Layered Design

```
API Layer (RogueMap, RogueList, RogueSet, RogueQueue)
    ↓
Index Layer (key → address mapping, or position tracking)
    ↓
Storage Engine (read/write byte data)
    ↓
Memory Allocator (MmapAllocator)
    ↓
UnsafeOps (sun.misc.Unsafe for direct memory access)
    ↓
Memory-Mapped Files (persistent or temporary)
```

### Data Structures

**RogueMap<K,V>** - Key-value store:
- `RogueMap.mmap().temporary()` - Temporary file mode (auto-deleted on JVM exit)
- `RogueMap.mmap().persistent(path)` - Persistent file mode (data survives restart)
- Index options: `basicIndex()`, `segmentedIndex(64)`, `primitiveIndex()`, `lowHeapIndex()`
- `forEach(BiConsumer<K,V>)` - Iterate over all key-value pairs
- TTL support: `defaultTTL(ttl, unit)` in builder; data stored as `[expireTime(8 bytes)][actual data]`
- Transactions: `beginTransaction()` returns AutoCloseable `Transaction<K,V>`

**RogueList<E>** - Doubly-linked list with O(1) random access:
- Maintains position index array for fast random access via `get(index)`
- Head/tail operations: `addFirst()`, `addLast()`, `removeFirst()`, `removeLast()`
- **Warning**: `addFirst()` and `removeFirst()` are O(n) due to position index shift; prefer `addLast()`/`removeLast()` for large lists
- Supports bidirectional iteration via `ListIterator<E>`

**RogueSet<E>** - Concurrent set:
- 64-segment design with StampedLock for high concurrency
- Optimistic read support for improved read performance
- Standard operations: `add()`, `contains()`, `remove()`
- `SetIterator` uses lazy segment loading (O(N/64) heap peak instead of O(N))
- Low-heap mode: `lowHeapIndex()` for String-key-only off-heap index

**RogueQueue<E>** - FIFO queue with two storage modes:
- **Linked mode** (unbounded): `RogueQueue.mmap().linked()`
- **Circular mode** (bounded): `RogueQueue.mmap().circular(capacity, maxElementSize)`
- Standard operations: `offer()`, `poll()`, `peek()`, `isFull()`
- LinkedQueue: snapshots head/tail/size to header on every offer/poll for crash recovery
- CircularQueue: recalculates count from headIdx/tailIdx on recovery

### Operations & Maintenance

**StorageMetrics** - Monitoring storage health:
- `getMetrics()` returns fragmentation ratio, used/available bytes, entry count, dead bytes
- `shouldCompact(threshold)` indicates when compaction is needed
- All four data structures support this API

**compact(allocSize)** - Space reclamation for persistent mode:
- Creates new file with only live data, eliminating fragmentation
- Returns new instance; old instance is closed
- Supported by RogueMap, RogueList, RogueSet, RogueQueue(linked)
- **Not supported**: temporary mode, CircularQueue

**checkpoint()** - Explicit crash recovery point:
- Forces index/metadata to disk for durable recovery
- Use when you need guaranteed recoverability between close() calls
- All four data structures support this in persistent mode

**AutoCheckpointManager** - Automatic checkpoint triggering:
- Time-interval mode: `autoCheckpoint(long interval, TimeUnit unit)` in builder
- Operation-count mode: `autoCheckpoint(int operationCount)` in builder

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bryan31/RogueMap](https://github.com/bryan31/RogueMap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
