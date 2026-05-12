---
trigger: always_on
description: This file is for AI agents working on EKV from a cold start.
---

# EKV Agent Context

This file is for AI agents working on EKV from a cold start.
It is not user-facing documentation.

Use it as the shortest accurate map of:
- current semantics
- safety invariants
- control-plane structure
- where the sharp edges are
- which tests to run before claiming a fix

Failure model in scope: non-malicious failures only.
Assume crash, restart, network partition, message delay/reordering, and operator mistakes.
Do not analyze Byzantine/malicious behavior unless explicitly asked.

## Read First
1. `lib/ekv.ex`
   Public API, return shapes, startup options, and user-visible semantics.
2. `lib/ekv/replica.ex`
   `_archdoc`, CASPaxos flow, sync/HWM logic, replication, GC, quorum, partition handling.
3. `lib/ekv/store.ex`
   SQLite schema, persisted metadata, stale-db checks, CAS/LWW persistence details.
4. `lib/ekv/supervisor.ex`
   Runtime mode split, scoped `:pg`, startup gates, blue-green handoff, persisted `node_id`.
5. `README.md` and `OPERATORS.md`
   Must stay aligned with real behavior.

## Current Product Model
- EKV is mixed-consistency.
- Default operations are eventual/LWW.
- CAS writes plus `consistent: true` reads provide per-key linearizable semantics when quorum is available.
- Modes are per key, not per store:
  - `LWW -> CAS` is allowed.
  - `CAS -> LWW` writes are rejected with `{:error, :cas_managed_key}`.
- `LWW -> CAS` is an operational migration, not a partition-safe fenced mode switch.
- A stale/partitioned node that has not yet learned CAS ownership for a key can still accept an eventual write for that key during cutover.
- Eventual reads on CAS-managed keys are still allowed.
- `consistent: true` is a barrier read, not a fast-path heuristic.

## Runtime Modes

### `mode: :member`
- Durable node.
- Runs SQLite shards, replication, GC, CAS proposer/acceptor logic.
- May run:
  - `wait_for_quorum`
  - `anti_entropy_interval`
  - `shutdown_barrier`
  - `blue_green`
  - `wire_compression_threshold`

### `mode: :client`
- Stateless node.
- Does not run SQLite, replicas, GC, sync, or blue-green machinery.
- Uses the same public API by routing to members.
- Supports:
  - `wait_for_route`
  - `wait_for_quorum` (via selected member)
  - `shutdown_barrier`
  - `wire_compression_threshold` config is accepted for shared child-spec simplicity, but
    current wire compression is member-to-member only.

Important:
- Client mode rejects member-only options like `:blue_green`, `:cluster_size`, `:node_id`, `:data_dir`, `:shards`, `:partition_ttl_policy`.

## CAS Configuration Reality
- CAS requires `cluster_size`.
- Member mode always resolves a stable `node_id` from disk, config, or first-boot random generation.
- CAS uses that same stable `node_id`; callers do not need to pass one explicitly unless they want to control the logical identity.
- Persisted `node_id` on disk wins over a conflicting configured `node_id`.
- Quorum math is by distinct logical `node_id`, not Erlang node name.

## Public API Contracts That Must Not Drift

### Eventual writes
- `EKV.put/4` eventual path: `:ok` or `{:error, :cas_managed_key}`
- `EKV.delete/3` eventual path: `:ok` or `{:error, :cas_managed_key}`

### CAS writes
- `EKV.put/4` CAS path:
  - `{:ok, vsn}`
  - `{:error, :conflict}`
  - `{:error, :unconfirmed}`
  - `{:error, :unavailable}` only when `resolve_unconfirmed: true` and the barrier resolution read cannot complete
  - operational failures may also surface as `{:error, :no_quorum}`, `{:error, :quorum_timeout}`, `{:error, :cluster_overflow}`, `{:error, :shutting_down}`, `{:error, :cas_not_configured}`
- `EKV.delete/3` CAS path:
  - same error model as CAS put
  - success shape is `{:ok, vsn}`
- `EKV.update/4`:
  - `{:ok, new_value, vsn}`
  - same error model as CAS put/delete

### Reads
- `EKV.get/2` is eventual.
- `EKV.lookup/2` is eventual and returns vsn.
- `EKV.get(name, key, consistent: true)` is a barrier/linearizable read for the key.
  - it returns `value | nil`
  - it raises if the consistent read itself cannot complete

### Streams
- `EKV.scan/2` yields `{key, value, vsn}`
- `EKV.keys/2` yields `{key, vsn}`
- In client mode these are still local Elixir streams, backed by paged RPC.

### `resolve_unconfirmed`
- Current default is `false`.
- If enabled on CAS writes, EKV does one internal barrier read on ambiguous accept-phase failure and resolves to:
  - the original success if the committed state matches the attempted write
  - `{:error, :conflict}` if it does not
  - `{:error, :unavailable}` if resolution itself fails

## Control Plane

### Scoped `:pg` is mandatory
- Do not use the default global `:pg` scope for EKV runtime behavior.
- Each EKV instance owns its own `:pg` scope via `EKV.Supervisor.pg_scope(name)`.
- All routing, distributed subscriptions, and shutdown coordination must use that scoped mesh.
- This isolation matters because multiple EKV instances can coexist on the same cluster.

### Member presence
- Ready members advertise themselves in scoped `:pg` region groups:
  - `{:ekv_members, name, region}`
- This is pinned by `EKV.MemberPresence`.
- New clients should discover members through this path, not by raw `Node.list/0`.

### Wire compression
- `wire_compression_threshold` defaults to `256 * 1024`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chrismccord/ekv](https://github.com/chrismccord/ekv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
