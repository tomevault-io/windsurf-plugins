---
trigger: always_on
description: Tango (**Ta**rget A**n**alyzer in **Go**) is a standalone library and service that fetches and compares Bazel target graphs across revisions of a repository. It answers two related questions:
---

# Tango (Target Analyzer) Repository Guide

## Key Concepts

Tango (**Ta**rget A**n**alyzer in **Go**) is a standalone library and service that fetches and compares Bazel target graphs across revisions of a repository. It answers two related questions:

1. **What does the target graph look like at a given revision?**
2. **Which targets changed between two revisions, and what is each changed target's BFS distance from the nearest direct cause?**

It is designed to run independently of the monorepo it analyzes — the only inputs are a remote URL, a base SHA, and optionally a set of change requests (PR/diff URLs + commit SHAs) to layer on top.

### Core design properties

1. **Content identity and deterministic computation** — the materialized git treehash identifies source content, while cache keys also include the computation-affecting inputs represented by each artifact's key schema. Canonical change URIs pin change requests to exact commits. Any new input that can change computed output requires a cache-key review; output-only options stay outside cache keys and are applied while sending.
2. **Value-oriented identities and safe handoffs** — repository identities and computed results should be treated as immutable after they are handed to another layer or goroutine. Tango still uses mutable workspaces, graph builders, slices, and maps internally; copy mutable collections before normalization or concurrent use when the caller may retain them.
3. **Scoped eventual consistency** — eventual consistency applies to the asynchronous, best-effort compared-target cache. Graph and treehash writes that occur while storing a computed graph are synchronous and request-bound, so they must succeed before that graph is returned.
4. **Streaming, chunked responses** — target graphs and change results are split into chunks to stay within gRPC per-message limits. Metadata mappings (target IDs → names, rule types, tags, attributes) may also span multiple chunks; consumers must merge them before use.
5. **ID-mapped payloads** — over the wire, targets reference each other (and their rule types, tags, attributes) by `int32` IDs into per-stream metadata maps. Comparison code re-maps both inputs into a canonical per-call namespace and prunes unreferenced metadata entries before sending. IDs are not guaranteed to be consistent across multiple target graphs.
6. **Always-on cancellation** — both request-bound and application-bound cancellation signals are honored. Long-running loops whose iteration count can be large check `ctx.Err()` periodically. A client disconnect cancels request work, while application shutdown cancels both request work and application-lifetime background work.

## Architecture

### Project Layout

```
tango/                              # repo root (Go module github.com/uber/tango)
├── proto/                          # Proto definitions (.proto files)
├── tangopb/                        # Generated proto code (committed)
│   └── tangopbmock/                # Generated mocks for YARPC server interfaces
├── controller/                     # YARPC service implementation (business logic, transport-adjacent)
├── orchestrator/                   # Cross-component coordinator: workspace lease, checkout, graph compute, cache I/O
│   └── orchestratormock/
├── graphrunner/                    # Strategy-pluggable target-graph computation
│   └── mock/
├── entity/                         # Domain value types (BuildDescription, ChangedTargets, TargetGraph, etc.)
├── mapper/                         # Proto-to-entity and entity-to-proto conversion
├── internal/                       # Internal-only packages (not importable by external consumers)
│   ├── mapper/                     # Internal mapping helpers
│   ├── streaming/                  # Chunked stream assembly
│   ├── targetdiff/                 # Per-target change classification
│   └── url/                        # URL normalization and hashing
├── config/                         # YAML config parsing and validation (storage, service, repository)
├── core/                           # Reusable infrastructure with no domain dependencies
│   ├── git/                        # Git CLI wrapper (clone, fetch, checkout, rev-parse, ...)
│   ├── repomanager/                # Per-repo worker-pool / clone manager
│   ├── storage/                    # Blob storage interface and impls (in-memory, disk)
│   ├── workspace/                  # Workspace abstraction over a git checkout, request application
│   ├── cachekey/                   # Cache path/key construction for graphs, treehashes, compared-target results
│   ├── errors/                     # TangoError type and ErrorCode classification
│   └── ...                         # bazel, execcmd, itg, targethasher, ...
├── observability/                  # Cross-cutting observability
│   └── metrics/                    # Tally-backed metrics emitter, Begin/Complete lifecycle, bucket definitions
├── integration/                    # Integration tests and benchmarks (requires a local repo)
├── docs/                           # Supplementary documentation (error taxonomy, observability notes)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uber/tango](https://github.com/uber/tango) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
