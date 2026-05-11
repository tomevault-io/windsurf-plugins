---
trigger: always_on
description: Distributed process registry + process groups + lifecycle monitoring + isolated subclusters.
---

# Group — CLAUDE.md

## What is Group

Distributed process registry + process groups + lifecycle monitoring + isolated subclusters.

## Project Structure

```
lib/
  group.ex              — Public API: register, join, members, monitor, dispatch, connect/disconnect
  group/event.ex        — %Group.Event{} struct
  group/supervisor.ex   — Top-level supervisor (rest_for_one: Data → Replica.Supervisor → Registry → ClusterLease)
  group/cluster_lease.ex — Local named-cluster TTL sweeper
  group/replica/
    data.ex             — GenServer that owns ETS tables. Pure function API for all ETS ops.
    supervisor.ex       — one_for_one supervisor for Replica shards
  group/replica.ex      — Sharded GenServer: replication, peer discovery, conflict resolution, monitoring
  group/application.ex  — Empty app supervisor (Group instances are started by consumers)
test/
  test_helper.exs       — Starts distribution (Node.start), disables prevent_overlapping_partitions
  group_test.exs        — Local tests (async: true)
  distributed_test.exs  — Multi-node tests using OTP :peer
  support/
    test_cluster.ex     — Peer node helpers (start_peers, spawn_register, spawn_join, etc.)
    test_conflict_resolver.ex — Custom resolver for tests
priv/bench/             — Benchmarks (run_local.sh, run_distributed.sh)
```

## Running Tests

```bash
mix test           # all tests
mix test test/group_test.exs           # local only
mix test test/distributed_test.exs     # distributed only
```

Tests require `elixirc_paths(:test)` includes `test/support/`. Distributed tests use OTP `:peer` module for real Erlang nodes.

## Running Benchmarks

```bash
cd priv/bench && ./run_local.sh
cd priv/bench && ./run_distributed.sh
```

## Architecture

### Supervision Tree

```
Group.Supervisor (rest_for_one)
├── Group.Replica.Data        — owns all ETS tables, survives shard crashes
├── Group.Replica.Supervisor  — one_for_one, N shard GenServers
│   ├── Replica shard 0
│   ├── Replica shard 1
│   └── ...
├── Registry (Elixir)         — :duplicate, for monitor subscriptions
└── Group.ClusterLease        — local named-cluster TTL sweeper
```

`rest_for_one` means: if Data dies, Replica.Supervisor restarts (rebuilds monitors from surviving ETS). If Replica.Supervisor dies, Registry restarts (monitors re-subscribe).

### Sharding

`phash2({cluster, key}, num_shards)` routes to shard. Default 8 shards. Must match across all nodes (validated on peer_connect). Including cluster in hash avoids false contention between default and named cluster operations.

### ETS Tables (per shard × 4 + 3 shared)

| Table | Type | Key | Tuple |
|-------|------|-----|-------|
| `reg_by_key` | `:set` | `{cluster, key}` | `{{cluster, key}, pid, meta, time, node}` |
| `reg_by_pid` | `:ordered_set` | `{pid, cluster, key}` | `{{pid, cluster, key}, meta, time, node}` |
| `pg_by_key` | `:ordered_set` | `{cluster, key, pid}` | `{{cluster, key, pid}, meta, time, node}` |
| `pg_by_pid` | `:ordered_set` | `{pid, cluster, key}` | `{{pid, cluster, key}, meta, time, node}` |
| `cluster_nodes` | `:bag` | cluster | `{cluster, node}` |
| `node_clusters` | `:bag` | node | `{node, cluster}` |
| `cluster_leases` | `:set` | cluster | `{cluster, ttl_ms, expires_at}` |

**Why ordered_set for by_pid tables**: Contiguous range scans for `entries_by_pid` and `delete_all_for_pid` (process death cleanup). Also enables efficient existence checks with `select(..., 1)`.

**Why ordered_set for pg_by_key**: `pg_members/4` scans `{cluster, key, *}` as a contiguous range — O(members in group), not O(table).

**Why set for reg_by_key**: Only needs direct lookup/delete by `{cluster, key}` — O(1).

All tables: `:public`, `read_concurrency: true`, `decentralized_counters: true`. No `write_concurrency` — writes serialize through shard GenServer anyway.

### Reads vs Writes

- **Reads** (`lookup`, `members`, `local_registry_count`) go directly to ETS — no GenServer involved
- **Writes** (`register`, `join`, `leave`, `unregister`) go through the shard's
  local request lane (`send` + monitor + tagged reply), not `GenServer.call`
- **Replication** arrives as `handle_info` messages on shard GenServers

### Config

Stored in `persistent_term` keyed by `{Group, name}`. Map with: `num_shards`, `log`, `callbacks`, optionally `extract_meta`, `resolve_registry_conflict`.

## Key Protocols

### Peer Discovery (per-shard, on nodeup/init)

1. Each shard sends `{:peer_connect, pid, shard, num_shards, clusters}` to its counterpart
2. Receiver adds sender to nil cluster ETS, computes shared clusters, sends `{:peer_connect_ack, ...}`
3. Both sides send `{:cluster_state, cluster, reg_data, pg_data}` for each shared cluster
4. `merge_remote_cluster_data` applies data: new entries insert, conflicts go through `resolve_conflict`

### Replication (steady state)

After discovery, writes replicate in two stages:
- nil cluster: uses `state.remote_shards` map
- Named clusters: uses `cluster_nodes` ETS table
- Sender batches: `replicate_registry_batch`, `replicate_pg_batch`
- Receiver buffers registry and PG lanes separately, bulk-applies ETS writes,
  then takes a bounded fairness turn for local work
- Remote shard sends use `send_nosuspend(..., [:noconnect])`; a `false` result

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phoenixframework/group](https://github.com/phoenixframework/group) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
