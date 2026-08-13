---
trigger: always_on
description: RaftTimeDB is a **transparent replication proxy** for SpacetimeDB. It wraps SpacetimeDB in a Raft consensus layer so that every write (reducer call) is replicated across multiple nodes. If any node dies, the cluster keeps running. Clients connect to RaftTimeDB exactly like they would connect to SpacetimeDB — same WebSocket protocol, same messages, no code changes.
---

# RaftTimeDB

## What This Is

RaftTimeDB is a **transparent replication proxy** for SpacetimeDB. It wraps SpacetimeDB in a Raft consensus layer so that every write (reducer call) is replicated across multiple nodes. If any node dies, the cluster keeps running. Clients connect to RaftTimeDB exactly like they would connect to SpacetimeDB — same WebSocket protocol, same messages, no code changes.

**Think of it like this:** SpacetimeDB is your database. RaftTimeDB makes it indestructible by running 3+ copies that stay perfectly in sync.

```
Your app (unchanged)  →  RaftTimeDB (any node)  →  SpacetimeDB
                              ↕ Raft consensus ↕
                         RaftTimeDB node 2  →  SpacetimeDB
                              ↕ Raft consensus ↕
                         RaftTimeDB node 3  →  SpacetimeDB
```

## For Users: How To Set Up and Use

### What You Need

- **3 machines** (or containers, or VMs) — Raft needs a majority, so 3 nodes tolerates 1 failure
- **SpacetimeDB** running on each machine (the same version)
- **Your module** published to ALL SpacetimeDB instances (see "Publishing Modules" below)
- **RaftTimeDB binary** on each machine (build with `cargo build --release`)

### Step 1: Start SpacetimeDB on Each Node

Each node needs its own SpacetimeDB instance:

```bash
# On each machine
spacetimedb-standalone start --listen-addr 0.0.0.0:3000
```

### Step 2: Publish Your Module to ALL Nodes

This is important: **publish the same module to every SpacetimeDB instance.**

```bash
spacetime publish --server http://node1:3000 my_module
spacetime publish --server http://node2:3000 my_module
spacetime publish --server http://node3:3000 my_module
```

RaftTimeDB replicates reducer calls (WebSocket writes), not module deployments (HTTP). Each SpacetimeDB instance needs the module installed independently. Once they all have the same module, Raft ensures they execute the same writes in the same order = identical state.

### Step 3: Start RaftTimeDB on Each Node

```bash
# Node 1
rafttimedb --node-id 1 --stdb-url ws://localhost:3000 \
  --peers "2=node2:4001,3=node3:4001"

# Node 2
rafttimedb --node-id 2 --stdb-url ws://localhost:3000 \
  --peers "1=node1:4001,3=node3:4001"

# Node 3
rafttimedb --node-id 3 --stdb-url ws://localhost:3000 \
  --peers "1=node1:4001,2=node2:4001"
```

Each node runs two servers:
- **:3001** — Client WebSocket (what your app connects to)
- **:4001** — Raft management API (inter-node + CLI)

### Step 4: Bootstrap the Cluster

Run this **once** to initialize Raft:

```bash
rtdb init --nodes 1=node1:4001 2=node2:4001 3=node3:4001
```

This triggers leader election. Within a few seconds, one node becomes the leader.

### Step 5: Verify

```bash
rtdb status --addr http://node1:4001
rtdb status --addr http://node2:4001
rtdb status --addr http://node3:4001
```

You should see one Leader and two Followers, all on the same term.

### Step 6: Connect Your App

Point your SpacetimeDB client at **any** RaftTimeDB node:

```
ws://node1:3001    # or node2:3001, or node3:3001 — any works
```

That's it. Your app works exactly like before, but now it's replicated.

### Docker Compose (Local Dev)

For local development, everything is preconfigured:

```bash
cd deploy/docker
docker compose up -d --build

# Wait for startup, then bootstrap:
curl -X POST http://localhost:4001/cluster/init \
  -H 'Content-Type: application/json' \
  -d '{"1":{"addr":"node-1:4001"},"2":{"addr":"node-2:4001"},"3":{"addr":"node-3:4001"}}'

# Check status:
curl http://localhost:4001/cluster/status
curl http://localhost:4002/cluster/status
curl http://localhost:4003/cluster/status

# Connect your client to ws://localhost:3001 (or :3002, :3003)
```

## Scaling the Cluster

RaftTimeDB supports **dynamic scaling** — add or remove nodes while the cluster is live. No downtime required.

### Adding Nodes

1. Start a new SpacetimeDB instance + RaftTimeDB proxy (with a unique `RTDB_NODE_ID`)
2. Publish the same module to the new SpacetimeDB instance
3. Add it to the Raft cluster:
```bash
rtdb add-node --node-id 4 --addr new-host:4001 --cluster http://existing-node:4001
```

The new node automatically syncs: it receives the Raft log from the leader, catches up, and becomes a voting member.

### Removing Nodes

**Always remove from Raft before stopping the container.** Never remove the current leader — remove followers first.

```bash
rtdb remove-node --node-id 4 --cluster http://existing-node:4001
# Then stop the node
```

### Quorum Rules

Always run an **odd number** of nodes for proper majority quorum:

| Nodes | Quorum needed | Tolerates failures | Use case |
|-------|--------------|-------------------|----------|
| 3     | 2            | 1                 | Dev / small prod |
| 5     | 3            | 2                 | Production |
| 7     | 4            | 3                 | High availability |

### Docker Compose Scaling

For local dev, each new node needs two docker-compose services added:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NodeNestor/RaftimeDB](https://github.com/NodeNestor/RaftimeDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
