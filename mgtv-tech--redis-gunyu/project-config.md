---
trigger: always_on
description: Redis GunYu is a Go 1.20 Redis data-management tool for real-time
---

# AGENTS.md

## Project Context

Redis GunYu is a Go 1.20 Redis data-management tool for real-time
synchronization, migration, RDB parsing and restore, local AOF storage,
checkpointed resume, filtering, and bidirectional synchronization.

The synchronization path supports standalone Redis, Redis Cluster, and mixed
source/target topologies. Replay modes are `sync`, `pipeline`, and `parallel`.
The documented consistency model is eventual/weak consistency; do not describe
the system as strongly consistent.

Redis 7.4.1 is the current qualified baseline. Redis 8 core compatibility has
been verified, but Redis 8 is not durability-qualified until the release soak
and benchmark gates have completed with retained reports.

## Architecture Map

| Path | Responsibility |
| --- | --- |
| `cmd/` | CLI commands, process lifecycle, and HTTP service entry points |
| `config/` | Configuration types, defaults, validation, CLI flags, and cloning |
| `syncer/` | Input/output orchestration, replay, resume, and bisync behavior |
| `pkg/rdb/` | Redis RDB format parsing and object decoding |
| `pkg/rdbrestore/` | RDB replay into standalone or cluster Redis |
| `pkg/store/` | Local AOF storage, rotation, readers/writers, and recovery |
| `pkg/redis/checkpoint/` | Checkpoint persistence and recovery state |
| `pkg/redis/client/cluster/` | Slot routing, transactions, MOVED/ASK handling, and topology refresh |
| `pkg/redis/keyspec/` | Command key extraction and routing metadata |
| `pkg/filter/` | Key, DB, command, and slot filtering |
| `pkg/cluster/` | GunYu HA control plane and leader election |
| `tests/nonbisync/` | Single-direction end-to-end regression suites |
| `tests/bisync/` | Bidirectional, fault, durability, performance, and Modules suites |
| `tests/integration/` | Repository-level real-Redis orchestration and regression runners |

The main data path is:

```text
Source Redis -> RedisInput/PSYNC -> channel and local storer
             -> replay -> RedisOutput -> Target Redis
```

## Critical Invariants

- Advance a checkpoint only after the corresponding data has been accepted by
  the intended processing or output stage. A newer checkpoint must never hide
  unapplied data.
- Restart and resume behavior must preserve valid offsets and business data.
  Pay particular attention to non-idempotent commands and writes made while a
  syncer is offline.
- Non-bisync runs must not create `redis-gunyu-bisync:*` metadata.
- Bisync must suppress mirrored writes from being propagated back indefinitely.
- `sync` and `pipeline`/`parallel` use different recovery metadata semantics.
  Do not interchange the meanings of `latest`, `frontier`, and `commit`.
- RDB full-sync metadata and AOF replay metadata represent different stages.
  Do not make transient RDB markers authoritative replay checkpoints.
- Cluster routing must use the actual command keys and slots. Preserve correct
  behavior across MOVED, ASK, failover, topology refresh, and slot migration.
- Validate synchronization using business values, checkpoint state, and bisync
  metadata invariants. Process health or equal key counts alone are insufficient.
- Redis commands unavailable on a source or target version require an explicit
  reject, skip, or downgrade policy. Do not silently claim compatibility.
- Changes to configuration fields must account for defaults, validation, clone
  behavior, CLI/YAML decoding, and tests.

For checkpoint key layout and recovery semantics, read
[`docs/checkpoint_en.md`](docs/checkpoint_en.md) before changing checkpoint or
bisync metadata code.

## Change-to-Test Matrix

Run the narrow package tests while iterating, then use the following minimum
regression scope before completion.

| Changed area | Minimum regression scope |
| --- | --- |
| Ordinary Go changes | `make test-static` and affected package tests |
| Shared behavior across packages | `go test ./... -count=1` |
| Concurrency, pipe, store, Redis, or syncer paths | `make test-race` |
| Redis client, role, or topology behavior | `make test-integration` |
| `pkg/rdb/**`, `pkg/rdbrestore/**`, `pkg/store/**`, or replay | Relevant unit tests, integration, bisync category3, nonbisync category3 |
| Checkpoint, resume, or bisync replay | `syncer`, `pkg/redis/checkpoint`, bisync category1/2/3 |
| `pkg/redis/client/cluster/**` | Cluster client tests with race, bisync category4/5, nonbisync category1/4/5 |
| `pkg/filter/**` or `pkg/redis/keyspec/**` | Filter/keyspec tests, bisync category4, nonbisync category7 |
| Configuration, authentication, or TLS | `config`, Redis client tests, security matrix |
| `pkg/cluster/**` or control plane | Cluster integration, nonbisync category10, both etcd runners |
| Redis Modules | Keyspec and RDB tests, bisync category10, nonbisync category11 |
| API, status, restart, or process lifecycle | `cmd`, `pkg/api`, `syncer`, and a restart/resume E2E case |
| Metrics or performance-sensitive replay | Relevant unit tests, benchmark, and sync-delay threshold |
| Release compatibility | `make test-release`, upgrade/rollback, required soak tier, and benchmark |

Unified entry points are defined in the root `Makefile`:

```bash
make test-static
make test-coverage
make test-race
make test-race-all
make test-integration
make test-e2e-smoke
make test-nightly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mgtv-tech/redis-GunYu](https://github.com/mgtv-tech/redis-GunYu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
