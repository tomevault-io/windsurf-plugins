---
trigger: always_on
description: The sync module orchestrates data flow from sources to destinations using a highly concurrent, pull-based asynchronous architecture with sophisticated backpressure control, real-time progress tracking, and automatic OAuth token management.
---

# Airweave Sync Architecture - Deep Dive

## Overview

The sync module orchestrates data flow from sources to destinations using a highly concurrent, pull-based asynchronous architecture with sophisticated backpressure control, real-time progress tracking, and automatic OAuth token management.

## Core Architecture Principles

### 1. Pull-Based Concurrency Model
- **Worker Pool Pattern**: Uses `AsyncWorkerPool` with semaphore-controlled concurrency (default: 20 workers)
- **Pull vs Push**: Workers pull entities from the stream only when ready, preventing system overload
- **Backpressure**: `AsyncSourceStream` uses bounded queues (default: 10000) to naturally throttle producers

### 2. Separation of Concerns
- **Producer/Consumer Decoupling**: Source generation runs independently from entity processing
- **Modular Pipeline**: Each stage (enrich, transform, vectorize, persist) is isolated
- **Resource Isolation**: Database sessions created only when needed to minimize connection usage

## Component Deep Dive

### SyncFactory
**Purpose**: Factory that builds SyncContext (data), SyncRuntime (services), and wires them into the orchestrator

**Key Responsibilities**:
- Builds SyncContext (frozen data) via SyncContextBuilder
- Builds source + cursor directly via `_build_source()` (uses SourceLifecycleService)
- Builds destinations via DestinationsContextBuilder
- Builds entity tracker via `_build_entity_tracker()` (inlined, no separate builder)
- Assembles SyncRuntime from per-sync state
- Configures contextual logging with sync metadata
- Wires pipelines, handlers, worker pool, and stream

**DI Model**: Instance-based with constructor-injected deps. Stateless app-scoped services (event_bus, usage_checker, processor, arf_service) are held by the factory and injected directly into consumers (SyncOrchestrator, EntityPipeline), not stored in SyncRuntime.

### SyncContext (frozen data)
**Purpose**: Immutable data describing a sync run. Inherits from `BaseContext` (sibling to `ApiContext`).

**Fields** (flat, no sub-contexts):
- `sync_id`, `sync_job_id`, `collection_id`, `source_connection_id`: Scope IDs
- `sync`, `sync_job`, `collection`, `connection`: Schema objects
- `execution_config`, `force_full_sync`, `batch_size`, `max_batch_latency_ms`: Config
- `entity_map`: Maps entity types to UUIDs
- `source_short_name`: Derived from source at build time
- From `BaseContext`: `organization`, `user`, `logger`

Can be passed directly as `ctx` to CRUD operations (it IS a `BaseContext`).

### SyncRuntime (live services)
**Purpose**: Holds per-sync mutable state for a sync run. Separate from SyncContext.

**Fields**:
- `source`: Source instance with OAuth token management
- `cursor`: Mutable sync cursor for incremental syncs
- `destinations`: List of destination instances
- `entity_tracker`: Centralized entity state tracker

Stateless singletons (event_bus, usage_checker, embedders) are NOT stored here — they are DI'd directly into their consumers via constructor injection.

Built by the factory, held by the orchestrator, injected into pipeline/handler constructors.

### Builders
- **SyncContextBuilder** (`builders/sync.py`): Builds data-only SyncContext
- **SyncFactory._build_source()**: Builds source + cursor directly (uses SourceLifecycleService), returns `SourceBuildResult`
- **DestinationsContextBuilder** (`builders/destinations.py`): Builds destination instances
- **SyncFactory._build_entity_tracker()**: Builds EntityTracker with initial counts (inlined into factory)

The factory calls build helpers in sequence, then assembles SyncRuntime.

### SyncOrchestrator
**Purpose**: Coordinates the entire sync workflow with error handling and progress tracking

**Workflow Stages**:
1. **Start**: Updates job status to IN_PROGRESS
2. **Process**: Manages entity streaming and concurrent processing
3. **Complete/Fail**: Updates final status with statistics

**Key Methods**:
- `run()`: Main entry point with try/catch for proper cleanup
- `_process_entities()`: Implements pull-based processing loop
- `_handle_completed_tasks()`: Cleans completed tasks and checks for errors
- `_wait_for_remaining_tasks()`: Ensures all tasks complete before finishing

**Constructor-Injected Services**: Receives `event_bus`, `usage_checker`, `usage_ledger`, and `sync_cursor_service` directly via constructor — not through SyncRuntime.

**Concurrency Management**:
```python
# Workers pull entities only when ready
async for entity in stream.get_entities():
    if entity.airweave_system_metadata.should_skip:
        # Skip without using a worker
        await sync_context.entity_tracker.record_skipped(1)
        continue

    # Submit to worker pool (blocks if all workers busy)
    task = await worker_pool.submit(...)
```

### AsyncSourceStream
**Purpose**: Manages async streaming with backpressure between producer and consumer

**Architecture**:
- **Producer Task**: Runs independently, filling queue from source generator
- **Bounded Queue**: Implements backpressure (blocks producer when full)
- **Consumer Interface**: `get_entities()` yields items as they become available

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [airweave-ai/airweave](https://github.com/airweave-ai/airweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
