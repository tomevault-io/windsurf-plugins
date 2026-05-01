---
trigger: always_on
description: ⛔ MANDATORY: Read this ENTIRE file before writing ANY SpacetimeDB Rust code. Contains SDK patterns from official documentation.
---


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
// ❌ WRONG — partial update or using .update() directly on table
ctx.db.user().update(User { name: Some("new".into()), ..Default::default() });

// ✅ CORRECT — find existing, spread it, update via primary key accessor
if let Some(user) = ctx.db.user().identity().find(ctx.sender) {
    ctx.db.user().identity().update(User { name: Some("new".into()), ..user });
}
```

### 5. Wrong Reducer Return Type

```rust
// ❌ WRONG — returning data from reducer
#[reducer]
pub fn get_user(ctx: &ReducerContext, id: Identity) -> Option<User> { ... }

// ❌ WRONG — mutable context
pub fn my_reducer(ctx: &mut ReducerContext, ...) { }

// ✅ CORRECT — reducers return Result<(), String> or nothing, immutable context
#[reducer]
pub fn do_something(ctx: &ReducerContext, value: String) -> Result<(), String> {
    if value.is_empty() {
        return Err("Value cannot be empty".to_string());
    }
    Ok(())
}
```

### 6. Wrong Client Connection Pattern

```rust
// ❌ WRONG — subscribing before connected
let conn = DbConnection::builder().build()?;
conn.subscription_builder().subscribe_to_all_tables();  // NOT CONNECTED YET!

// ✅ CORRECT — subscribe in on_connect callback
DbConnection::builder()
    .on_connect(|conn, identity, token| {
        conn.subscription_builder()
            .on_applied(|ctx| println!("Ready!"))
            .subscribe_to_all_tables();
    })
    .build()?;
```

### 7. Forgetting to Advance the Connection

```rust
// ❌ WRONG — connection never processes messages
let conn = DbConnection::builder().build()?;
// ... callbacks never fire ...

// ✅ CORRECT — must call one of these to process messages
conn.run_threaded();           // Spawn background thread
// OR
conn.run_async().await;        // Async task
// OR (in game loop)
conn.frame_tick()?;            // Manual polling
```

### 8. Missing Table Trait Import

```rust
// ❌ WRONG — "no method named `insert` found"
use spacetimedb::{table, reducer, ReducerContext};
ctx.db.user().insert(...);  // ERROR!

// ✅ CORRECT — import Table trait for table methods
use spacetimedb::{table, reducer, Table, ReducerContext};
ctx.db.user().insert(...);  // Works!
```

### 9. Wrong ScheduleAt Variant

```rust
// ❌ WRONG — At variant doesn't exist
scheduled_at: ScheduleAt::At(future_time),

// ✅ CORRECT — use Time variant
scheduled_at: ScheduleAt::Time(future_time),
```

### 10. Identity to String Conversion

```rust
// ❌ WRONG — to_hex() returns HexString<32>, not String
let id: String = identity.to_hex();  // Type mismatch!

// ✅ CORRECT — chain .to_string()
let id: String = identity.to_hex().to_string();
```

### 11. Timestamp Duration Extraction

```rust
// ❌ WRONG — returns Result, not Duration directly
let micros = ctx.timestamp.to_duration_since_unix_epoch().as_micros();

// ✅ CORRECT — unwrap the Result
let micros = ctx.timestamp.to_duration_since_unix_epoch()
    .unwrap_or_default()
    .as_micros();
```

### 12. Borrow After Move

```rust
// ❌ WRONG — `tool` moved into struct, then borrowed
ctx.db.stroke().insert(Stroke { tool, color, ... });
if tool == "eraser" { ... }  // ERROR: value moved!

// ✅ CORRECT — check before move, or use clone
let is_eraser = tool == "eraser";
ctx.db.stroke().insert(Stroke { tool, color, ... });
if is_eraser { ... }
```

### 13. Client SDK Uses Blocking I/O

The SpacetimeDB Rust client SDK uses blocking I/O. If mixing with async runtimes (Tokio, async-std), use `spawn_blocking` or run the SDK on a dedicated thread to avoid blocking the async executor.

### 14. Wrong Schedule Syntax
```rust
// ❌ WRONG — `schedule` is not a valid table type
#[table(name = tick_timer, schedule(reducer = tick, column = scheduled_at))]

// ✅ CORRECT — `scheduled` is a valid table type
#[table(name = tick_timer, scheduled(reducer = tick, column = scheduled_at))]
```
---

## 1) Common Mistakes Table

### Server-side errors

| Wrong | Right | Error |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lethalchip/SpaceChatDB](https://github.com/Lethalchip/SpaceChatDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
