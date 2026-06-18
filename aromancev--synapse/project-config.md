---
trigger: always_on
description: Associative structured local memory store. Use to remember and recall anything and retrieve linked memories.
---


# Synapse

Use the `synapse` CLI as a structured local memory and graph store.

## Core model

Synapse has three main concepts:

1. **Schemas**
   - runtime JSON Schemas
   - define the allowed structure of node payloads
   - archived schemas remain in history

2. **Nodes**
   - typed records validated against a schema
   - have payload, keywords, and archive state
   - payload updates replace the entire payload

3. **Links**
   - connections between node pairs
   - links are bidirectional
   - used for graph traversal

IDs are prefixed and human-readable:

- `schema_<...>`
- `node_<...>`
- `event_<...>`

## Command map

### Initialize

```bash
synapse init
```

Use `--db-path` to point at a non-default SQLite file:

```bash
synapse --db-path /path/to/synapse.db init
```

Default DB path is `.synapse/db`.

### Config

Read current config:

```bash
synapse config get
```

Update config from JSON argument or stdin:

```bash
cat config.json | synapse config update
```

Print config schema:

```bash
synapse config schema
```

### Schemas

Add a schema:

```bash
cat schema.json | synapse schemas add --name task
```

List schemas:

```bash
synapse schemas list
synapse schemas list --archived
```

Archive a schema:

```bash
synapse schemas archive <schema-id>
```

### Nodes

Add a node:

```bash
cat payload.json | synapse nodes add --schema-id <schema-id>
```

List nodes for a schema:

```bash
synapse nodes list --schema-id <schema-id>
synapse nodes list --schema-id <schema-id> --archived
```

Update a node payload:

```bash
cat payload.json | synapse nodes update <node-id>
```

Archive a node:

```bash
synapse nodes archive <node-id>
```

Search node IDs:

```bash
synapse nodes search '{"query":["keyword phrase"]}' --limit 20
```

### Keywords

Read node keywords:

```bash
synapse nodes keywords get <node-id>
```

Replace node keywords:

```bash
cat keywords.json | synapse nodes keywords update <node-id>
```

### Links and graph

Create or remove links:

```bash
synapse links add <from-node-id> <to-node-id>
synapse links remove <from-node-id> <to-node-id>
```

Traverse from explicit node IDs:

```bash
synapse graph get '["<node-id>"]' --depth 2 --breadth 10
```

Search first, then traverse:

```bash
synapse graph search '{"query":["keyword phrase"]}' --search-limit 5 --depth 2 --breadth 10
```

### Projections and replication

Run projections explicitly:

```bash
synapse projections run
```

Run replication explicitly:

```bash
synapse replication run
```

Restore from configured replicator into an empty event store:

```bash
synapse replication restore
```

## JSON input and query shapes

Commands that accept JSON can read it either:

- as a positional argument
- from `stdin`

Structured query commands use JSON too. `query` is always an array: each array element is OR-ed, and the space-separated words inside one element are AND-ed.

Examples:

```bash
synapse schemas add --name note '{"type":"object","properties":{"title":{"type":"string"}},"required":["title"]}'
synapse nodes add --schema-id "$SCHEMA_ID" '{"title":"Write README","status":"todo"}'
synapse nodes update "$NODE_ID" '{"title":"Write README","status":"done"}'
synapse nodes search '{"query":["readme"]}'
synapse nodes search '{"query":["readme docs","ops"]}'
synapse graph get '["node_01..."]'
synapse graph search '{"query":["engineering"]}' --depth 2
synapse graph search '{"query":["engineering sqlite","golang"]}' --depth 2
```

`stdin` form:

```bash
cat schema.json | synapse schemas add --name note
cat payload.json | synapse nodes add --schema-id "$SCHEMA_ID"
cat payload.json | synapse nodes update "$NODE_ID"
cat keywords.json | synapse nodes keywords update "$NODE_ID"
```

## jq workflow patterns

Use `jq` heavily. Synapse returns JSON, and updates are full replacement.

### Filter nodes

```bash
synapse nodes list --schema-id "$TASK_SCHEMA_ID" \
  | jq -r '.[] | select(.payload.status == "todo") | .payload.title'
```

### Patch-like payload update

Do not send partial payloads unless the partial object is valid as the full payload.

Preferred pattern:

```bash
synapse nodes list --schema-id "$TASK_SCHEMA_ID" \
  | jq -c --arg id "$NODE_ID" '
      .[]
      | select(.id == $id)
      | .payload
      | .status = "done"
    ' \
  | synapse nodes update "$NODE_ID"
```

### Append keywords safely

```bash
synapse nodes keywords get "$NODE_ID" \
  | jq -c '. + ["new-keyword"] | unique' \
  | synapse nodes keywords update "$NODE_ID"
```

### Select due tasks

If task payloads use ISO dates:

```bash
TODAY=$(date +%F)

synapse nodes list --schema-id "$TASK_SCHEMA_ID" \
  | jq -r --arg today "$TODAY" '
      .[]
      | select(.payload.status != "done")
      | select(.payload.due != null and .payload.due <= $today)
      | .id
    '
```

## Usage best practice
The guidelines in this section are IMPORTANT. Make sure to follow them and help your user to follow them as well.

### Schemas
Each schema should have a top-level description (JSON schema `description` field) of how its nodes can be used and what situations they can be used in. For example, for a task schema the description can be like this:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aromancev/synapse](https://github.com/aromancev/synapse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
