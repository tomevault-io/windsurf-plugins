---
trigger: always_on
description: Upspeak is a personal-first, federated knowledge infrastructure designed to collect, organise, and synthesise data from web sources and your own inputs. It follows a modular, event-driven architecture built on domain-driven design principles.
---

# Claude Instructions for Upspeak

## Project Overview

Upspeak is a personal-first, federated knowledge infrastructure designed to collect, organise, and synthesise data from web sources and your own inputs. It follows a modular, event-driven architecture built on domain-driven design principles.

**Architecture:**
- **API-first**: Pure API server, no bundled UI. Clients connect over HTTP
- **Modular design**: Each module implements the `app.Module` interface for HTTP and message handlers
- **Hybrid sync core + NATS JetStream**: Synchronous writes to archive (SQLite + files), NATS JetStream for downstream events
- **Hexagonal architecture**: Domain layer (`core/`) separated from infrastructure (`archive/`, `nats/`)
- **NATS isolation**: All NATS code lives in `nats/` — no other package imports nats-io
- **Local/remote archive split**: `core.Archive` interface supports both local (SQLite + files) and remote (Postgres + object storage) implementations
- **Knowledge graph**: Nodes, Edges, Threads, and Annotations form a structured graph with UUID v7 identifiers and human-friendly short IDs
- **Filter engine**: Reusable condition sets with 15 operators, dot-path field resolution, and AND/OR modes
- **Job system**: Async job tracking via NATS JetStream JOBS stream with durable pull consumer

**Key packages:**
- `app/`: Micro-framework for composing modules, HTTP routing, and application lifecycle. NATS-unaware — receives Publisher/Subscriber interfaces via DI
- `core/`: Domain models (Node, Edge, Thread, Annotation, Filter, Job, User, Repository), Archive sub-interfaces, event types, identity system
- `archive/`: Local archive implementation (SQLite metadata + filesystem body storage). Implements `core.Archive`
- `nats/`: NATS JetStream infrastructure — embedded server, publisher, subscriber, stream lifecycle. Isolated from all other packages
- `repo/`: Repository CRUD and knowledge graph API module. Mounted at `/api/v1`
- `filter/`: Filter condition evaluation engine and filter CRUD module. Mounted at `/api/v1`
- `jobs/`: Job tracking, cancellation, and JetStream runner module. Mounted at `/api/v1`
- `api/`: Response envelope, HTTP helpers, middleware (ETag, RequestID)

## Critical Rules

1. **ALWAYS** follow patterns established in `app/` and `core/` packages
2. **ALWAYS** add GoDoc-style comments for all public functions and types
3. **ALWAYS** add comments for longer private methods (>20 lines)
4. **ALWAYS** write documentation in en-IN (Indian English: "organise", "behaviour", "colour")
5. **ALWAYS** make small commits per logical chunk of work, not monolithic batches
6. **NEVER** respond with summaries unless explicitly requested
7. **NEVER** skip error handling — check and handle all errors immediately
8. **NEVER** use `panic` for normal error conditions
9. **NEVER** create deep nesting — extract functions or use early returns
10. **NEVER** add repository directory structure to README — structure is ephemeral
11. **NEVER** put NATS imports in any package other than `nats/`

## Build Commands

```bash
# Build the binary
./build.sh build

# Development mode (requires upspeak.yaml)
./build.sh dev

# Clean artifacts
./build.sh cleanup

# Run tests
go test ./...
```

## Identity System

All entities use **UUID v7** as primary key (time-ordered, via `google/uuid`). Each entity also carries a **short ID** — a human-friendly `{PREFIX}-{SEQ}` identifier:

- `REPO-1`, `NODE-42`, `EDGE-15`, `THREAD-7`, `ANNO-3`, `FILTER-2`, `JOB-109`
- Short ID sequences are scoped: per-repo (nodes, edges, threads, annotations), per-user (repos), or global (jobs, schedules, users)
- `core.NewID()` generates a UUID v7. `core.FormatShortID(prefix, seq)` formats a short ID
- `core.ParseShortID(s)` extracts prefix and sequence number
- Sequence generation is internal to `archive/` — not exposed through `core.Archive`

## Archive Interface

`core.Archive` is composed of sub-interfaces that both local and remote implementations can satisfy:

```go
type Archive interface {
    RepositoryStore   // SaveRepository, GetRepository, ListRepositories, DeleteRepository, slug management
    NodeStore         // SaveNode, SaveBatchNodes, GetNode, DeleteNode, ListNodes, GetNodeEdges, GetNodeAnnotations
    EdgeStore         // SaveEdge, SaveBatchEdges, GetEdge, DeleteEdge, ListEdges
    ThreadStore       // SaveThread, GetThread, DeleteThread, ListThreads, AddNodeToThread, RemoveNodeFromThread
    AnnotationStore   // SaveAnnotation, GetAnnotation, DeleteAnnotation, ListAnnotations
    FilterStore       // SaveFilter, GetFilter, DeleteFilter, ListFilters, GetFilterReferences
    JobStore          // SaveJob, GetJob, GetJobByShortID, ListJobs
    RefResolver       // ResolveRef — resolves short ID or UUID to (uuid, entityType, error)
}
```

**Local archive storage split:**
- **Metadata** (SQLite): type, subject, content_type, edges, config — everything queryable
- **Node body content** (filesystem): stored at `{archive_path}/content/{node_id}` as files
- This mirrors the high-level architecture: local = SQLite + files, remote = Postgres + object storage


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [upspeak/upspeak](https://github.com/upspeak/upspeak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
