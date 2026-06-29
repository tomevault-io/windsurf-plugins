---
trigger: always_on
description: Notes for AI agents working on this codebase.
---

# Agent Notes

Notes for AI agents working on this codebase.

## Project overview

Redix is a low-level Redis/Valkey driver for Elixir. It supports single-node connections,
Sentinel-based failover, PubSub, and Redis Cluster. The philosophy is minimal and
composable: build on `Redix` connections (one per node), use ETS for fast lookups,
mirror the `Redix` API in higher-level modules.

## Codebase conventions

- **gen_statem over GenServer** for stateful processes. `Redix.Connection`,
  `Redix.PubSub.Connection`, and `Redix.Cluster.Manager` all use `:gen_statem`.
  Prefer `:state_functions` callback mode when there are meaningful states.
- **NimbleOptions** for option validation. See `Redix.StartOptions` and the
  `@start_link_opts_schema` in `Redix.Cluster`.
- **Telemetry only, no Logger** in library code. All observability goes through
  `:telemetry.execute/3`. The `Redix.Telemetry` module documents events and provides
  a default handler that logs them. When adding new functionality, add telemetry events
  rather than Logger calls.
- **Typespecs** on all public functions.
- **`@moduledoc false`** on internal modules (Manager, Hash, CommandParser, etc.).
- **`!` variants** for all public command functions (raises instead of returning error tuples).
- Tests use `start_supervised!` and unique names to avoid collisions in async tests.
  Cluster integration tests use `@moduletag :cluster` and skip when the Docker cluster
  isn't available.

## Redis Cluster implementation

### Architecture

```
Redix.Cluster (Supervisor, public API)
├── Registry (unique keys, node_id -> Redix pid)
├── DynamicSupervisor (supervises Redix connections)
├── Task.Supervisor (parallel pipeline execution)
└── Redix.Cluster.Manager (gen_statem: topology + connection lifecycle)
    ├── ETS: slot_table (slot 0..16383 -> {primary_id, [replica_id]})
    └── ETS: command_cache (command name -> key spec, for commands outside the table)
```

All resource names are derived deterministically from the cluster name:
`:my_cluster` -> `:"my_cluster_slots"`, `:"my_cluster_command_cache"`,
`:"my_cluster_registry"`, `:"my_cluster_manager"`, `:"my_cluster_pool"`,
`:"my_cluster_task_supervisor"`.
This eliminates the need for `persistent_term` or any external lookup.

### Key modules

- **`Redix.Cluster`** (`lib/redix/cluster.ex`) — Public API + Supervisor. `command/3`,
  `pipeline/3`, `transaction_pipeline/3`. Handles MOVED/ASK redirections, transparent
  pipeline splitting across nodes. The `:name` option is required.

- **`Redix.Cluster.Manager`** (`lib/redix/cluster/manager.ex`) — gen_statem with three
  states: `:disconnected` (no topology yet, async connect), `:ready`, and
  `:cooling_down`. Manages topology via `CLUSTER SLOTS`, starts
  Redix connections registered in the Registry via
  `{:via, Registry, {registry, node_id, role}}` where `role` is `:primary` or
  `:replica` (the Registry *value*). Uses named timeout `{:timeout, :periodic_refresh}`
  for periodic refresh and `:state_timeout` for the 1-second cooldown after reactive
  refreshes.

- **`Redix.Cluster.Hash`** (`lib/redix/cluster/hash.ex`) — CRC16-XMODEM with
  compile-time lookup table, hash tag extraction, `hash_slot/1`.

- **`Redix.Cluster.CommandParser`** (`lib/redix/cluster/command_parser.ex`) — Static
  lookup table (~180 commands) mapping command names to first key position. Handles
  EVAL/EVALSHA numkeys parsing, XREAD/XREADGROUP STREAMS keyword scanning, and the
  position-2 commands OBJECT/BITOP. Returns `{:ok, key}`, `:no_key`, or `:unknown`
  (the last resolved at runtime by `Redix.Cluster` via COMMAND INFO/GETKEYS).

### Design decisions

- **Registry for connections, ETS for slots.** The slot table is pure data (16384 entries
  mapping to `{primary_id, [replica_id]}`) — ETS is the right tool. The connection map
  (node_id -> pid) is process registration — Registry handles auto-cleanup on process
  death and supports `:via` tuples for transparent naming. The Registry *value* records
  the node's `role` so keyless commands route to primaries (`get_random_connection/1`)
  and replica lookups stay separate from primary lookups.

- **Replica reads are opt-in** (`read_from_replicas: true`). When off (default), only
  primaries are connected and the slot table stores `[]` for replicas — behavior is
  identical to primary-only. When on, the Manager also connects/supervises/monitors one
  connection per replica `host:port`, passing `readonly: true` so the connector issues
  `READONLY` after *every* (re)connect (it lives in `Redix.Connector.auth_and_select`
  alongside AUTH/SELECT, so reconnects redo it automatically). Per-call routing is the
  `:route` option on `command/3`/`pipeline/3` (`:primary` | `:replica` | `:prefer_replica`),
  resolved in `Redix.Cluster.resolve_connection/4`. `transaction_pipeline/3` only allows
  `:primary` (MULTI/EXEC must run on the primary). A write mistakenly routed to a replica
  comes back as `MOVED` and is followed to the primary by the existing redirect machinery.

- **Node roles are reconciled on every topology refresh.** `ensure_connections/2`
  compares each live connection's registered role (the Registry *value*) against the
  role the latest `CLUSTER SLOTS` assigns it, and terminates+restarts on a mismatch

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whatyouhide/redix](https://github.com/whatyouhide/redix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
