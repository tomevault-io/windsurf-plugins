---
trigger: always_on
description: **If you are migrating existing SpacetimeDB 1.0 code to 2.0, apply `spacetimedb-migration-2.0.mdc` first.** It documents breaking changes (reducer callbacks → event tables, `name`→`accessor`, `sender()` method, etc.) and should be considered before other rules.
---


# SpacetimeDB Rules (All Languages)

## Migrating from 1.0 to 2.0?

**If you are migrating existing SpacetimeDB 1.0 code to 2.0, apply `spacetimedb-migration-2.0.mdc` first.** It documents breaking changes (reducer callbacks → event tables, `name`→`accessor`, `sender()` method, etc.) and should be considered before other rules.

---

## Language-Specific Rules

| Language | Rule File |
|----------|-----------|
| **TypeScript/React** | `spacetimedb-typescript.mdc` (MANDATORY) |
| **Rust** | `spacetimedb-rust.mdc` (MANDATORY) |
| **C#** | `spacetimedb-csharp.mdc` (MANDATORY) |
| **Migrating 1.0 → 2.0** | `spacetimedb-migration-2.0.mdc` |

---

## Core Concepts

1. **Reducers are transactional** — they do not return data to callers
2. **Reducers must be deterministic** — no filesystem, network, timers, or random
3. **Read data via tables/subscriptions** — not reducer return values
4. **Auto-increment IDs are not sequential** — gaps are normal, don't use for ordering
5. **`ctx.sender` is the authenticated principal** — never trust identity args

---

## Feature Implementation Checklist

When implementing a feature that spans backend and client:

1. **Backend:** Define table(s) to store the data
2. **Backend:** Define reducer(s) to mutate the data
3. **Client:** Subscribe to the table(s)
4. **Client:** Call the reducer(s) from UI — **don't forget this step!**
5. **Client:** Render the data from the table(s)

**Common mistake:** Building backend tables/reducers but forgetting to wire up the client to call them.

---

## Index System

SpacetimeDB automatically creates indexes for:
- Primary key columns
- Columns marked as unique

You can add explicit indexes on non-unique columns for query performance.

**Index names must be unique across your entire module (all tables).** If two tables have indexes with the same declared name → conflict error.

**Schema ↔ Code coupling:**
- Your query code references indexes by name
- If you add/remove/rename an index in the schema, update all code that uses it
- Removing an index without updating queries causes runtime errors

---

## Commands

```bash
# Login to allow remote database deployment e.g. to maincloud
spacetime login

# Start local SpacetimeDB
spacetime start

# Publish module
spacetime publish <db-name> --module-path <module-path>

# Clear and republish
spacetime publish <db-name> --clear-database -y --module-path <module-path>

# Generate client bindings
spacetime generate --lang <lang> --out-dir <out> --module-path <module-path>

# View logs
spacetime logs <db-name>
```

---

## Deployment

- Maincloud is the spacetimedb hosted cloud and the default location for module publishing
- The default server marked by *** in `spacetime server list` should be used when publishing
- If the default server is maincloud you should publish to maincloud
- Publishing to maincloud is free of charge
- When publishing to maincloud the database dashboard will be at the url: https://spacetimedb.com/@<username>/<database-name>
- The database owner can view utilization and performance metrics on the dashboard

---

## Debugging Checklist

1. Is SpacetimeDB server running? (`spacetime start`)
2. Is the module published? (`spacetime publish`)
3. Are client bindings generated? (`spacetime generate`)
4. Check server logs for errors (`spacetime logs <db-name>`)
5. **Is the reducer actually being called from the client?**

---

## Editing Behavior

- Make the smallest change necessary
- Do NOT touch unrelated files, configs, or dependencies
- Do NOT invent new SpacetimeDB APIs — use only what exists in docs or this repo
- Do NOT add restrictions the prompt didn't ask for — if "users can do X", implement X for all users




# SpacetimeDB Rust SDK

## ⛔ COMMON MISTAKES — LLM HALLUCINATIONS

These are **actual errors** observed when LLMs generate SpacetimeDB Rust code:

### 1. Wrong Crate for Server vs Client

```rust
// ❌ WRONG — using client crate for server module
use spacetimedb_sdk::*;  // This is for CLIENTS only!

// ✅ CORRECT — use spacetimedb for server modules
use spacetimedb::{table, reducer, Table, ReducerContext, Identity, Timestamp};
```

### 2. Wrong Table Macro Syntax

```rust
// ❌ WRONG — using attribute-style like C#
#[spacetimedb::table]
#[primary_key]
pub struct User { ... }

// ❌ WRONG — SpacetimeType on tables (causes conflicts!)
#[derive(SpacetimeType)]
#[table(accessor = my_table)]
pub struct MyTable { ... }

// ✅ CORRECT — use #[table(...)] macro with options, NO SpacetimeType
#[table(accessor = user, public)]
pub struct User {
    #[primary_key]
    identity: Identity,
    name: Option<String>,
}
```

### 3. Wrong Table Access Pattern

```rust
// ❌ WRONG — using ctx.Db or ctx.db() method or field access
ctx.Db.user.Insert(...);
ctx.db().user().insert(...);
ctx.db.player;  // Field access

// ✅ CORRECT — ctx.db is a field, table names are methods with parentheses
ctx.db.user().insert(User { ... });
ctx.db.user().identity().find(ctx.sender);
ctx.db.player().id().find(&player_id);
```

### 4. Wrong Update Pattern

```rust

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lethalchip/SpaceChatDB](https://github.com/Lethalchip/SpaceChatDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
