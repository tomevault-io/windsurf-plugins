---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DrasiLib is a Rust library for real-time data change processing that implements a reactive event-driven architecture. It processes data changes from various sources through Cypher queries and delivers results to reactions. This is a library-only project that can be used as a dependency by other applications.

## Key Architecture Components

### Core Abstractions
- **Sources**: Data ingestion points (PostgreSQL WAL, HTTP, gRPC, Mock, Platform via Redis Streams)
- **Queries**: Cypher-based continuous queries that process data changes
- **Reactions**: Output destinations (HTTP, gRPC, SSE, Log)
- **Routers**: Handle event routing between components (DataRouter, SubscriptionRouter, BootstrapRouter)
- **Channels**: Async communication between components using Tokio channels
- **Priority Queues**: Timestamp-ordered event queues with backpressure support
- **Bootstrap Providers**: Pluggable components for initial data delivery

### Channels and Backpressure

DrasiLib uses two dispatch modes for event routing:

#### Dispatch Modes

**Channel Mode (Default)** - Recommended for most use cases
- Creates **isolated MPSC channels per subscriber** (query or reaction)
- Provides **backpressure** when subscribers are slow - sources wait instead of dropping events
- **Zero message loss** with blocking enqueue to priority queues
- Slow subscribers don't affect fast ones
- Use this when: queries have different processing speeds, message loss is unacceptable

**Broadcast Mode**
- Uses **single shared broadcast channel** for all subscribers
- **No backpressure** - fast send, receivers can lag
- **Messages may be lost** when receivers fall behind
- Lower memory usage (one channel vs N channels)
- Use this when: all subscribers process at similar speeds, high fanout (10+ subscribers), can tolerate message loss

#### Priority Queue Backpressure

Priority queues support two enqueue strategies based on dispatch mode:

**Blocking Enqueue (`enqueue_wait()`)** - Used with Channel Mode
- Waits until space is available in the queue
- Never drops events - provides end-to-end backpressure
- Backpressure flows: Query Priority Queue → Channel Buffer → Source
- **Safe for Channel mode** (isolated channels)
- **Never use with Broadcast mode** (causes deadlock)

**Non-blocking Enqueue (`enqueue()`)** - Used with Broadcast Mode
- Returns immediately, drops events when queue is full
- Prevents deadlock in broadcast scenarios
- Metrics track `drops_due_to_capacity`

**Configuration:**
```yaml
# Channel mode (default) - backpressure enabled, zero message loss
sources:
  - id: my_source
    dispatch_mode: channel  # Default, no need to specify
    dispatch_buffer_capacity: 1000  # Per-subscriber channel buffer

queries:
  - id: my_query
    priority_queue_capacity: 10000  # Events queue before backpressure
    dispatch_mode: channel  # For query → reaction routing

# Broadcast mode - lower memory, possible message loss
sources:
  - id: high_fanout_source
    dispatch_mode: broadcast
    dispatch_buffer_capacity: 100000  # Large shared buffer
```

**Metrics:**
- `blocked_enqueue_count`: Times backpressure caused blocking (channel mode)
- `drops_due_to_capacity`: Events dropped (broadcast mode or overload)
- `current_depth`, `max_depth_seen`: Queue utilization
- `total_enqueued`, `total_dequeued`: Throughput tracking

#### Adaptive Batcher Channel Capacity

Adaptive reactions (HTTP, gRPC) and the HTTP source use an internal channel between the receiver task and the `AdaptiveBatcher`. This channel capacity **automatically scales** with the `max_batch_size` configuration:

**Automatic Scaling**: `channel_capacity = max_batch_size × 5`

This 5x multiplier provides:
- **Pipeline parallelism**: Next batch accumulates while current batch is being sent
- **Burst handling**: Absorbs temporary traffic spikes without backpressure
- **Throughput smoothing**: Reduces blocking on channel sends

**Scaling Examples**:
| max_batch_size | Channel Capacity | Memory (1KB/event) |
|----------------|------------------|---------------------|
| 100            | 500              | ~500 KB            |
| 1,000 (default)| 5,000            | ~5 MB              |
| 5,000          | 25,000           | ~25 MB             |

**Implementation**: See `AdaptiveBatchConfig::recommended_channel_capacity()` in `/Users/allenjones/dev/agentofreality/drasi/drasi-core/lib/src/utils/adaptive_batcher.rs`

### Bootstrap Provider Architecture
DrasiLib features a **universal pluggable bootstrap provider system** where ALL sources support configurable bootstrap providers, completely separating bootstrap (initial data delivery) from source streaming logic.

**Key Architectural Principle**: Bootstrap providers are independent from sources. Any source can use any bootstrap provider, enabling powerful use cases like "bootstrap from database, stream changes from HTTP endpoint."

#### All Sources Support Bootstrap Providers
- **PostgresReplicationSource**: ✅ Delegates to configured provider
- **HttpSource (Adaptive)**: ✅ Delegates to configured provider
- **GrpcSource**: ✅ Delegates to configured provider

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drasi-project/drasi-core](https://github.com/drasi-project/drasi-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
