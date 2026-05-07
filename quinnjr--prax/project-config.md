---
trigger: always_on
description: This project prioritizes performance. Follow these guidelines to write efficient, allocation-conscious code.
---

# Performance Optimization Guidelines

This project prioritizes performance. Follow these guidelines to write efficient, allocation-conscious code.

## Memory Optimization

### Use String Interning

For repeated identifiers (column names, table names):

```rust
use prax_query::mem_optimize::interning::{GlobalInterner, ScopedInterner};

// ✅ Good: Intern repeated field names
let interner = GlobalInterner::get();
let field = interner.intern("user_id"); // Shared across all uses

// ✅ Good: Use scoped interner for request-local interning
let mut scoped = ScopedInterner::new();
for column in &columns {
    let interned = scoped.intern(column);
    // Memory freed when scoped is dropped
}

// ❌ Bad: Repeated String allocations
for _ in 0..1000 {
    let field = "user_id".to_string(); // 1000 allocations!
}
```

### Use Arena Allocation

For query building with many temporary allocations:

```rust
use prax_query::mem_optimize::arena::QueryArena;

// ✅ Good: Arena-based query building
let arena = QueryArena::new();
let sql = arena.scope(|scope| {
    let filter = scope.and(vec![
        scope.eq("active", true),
        scope.or(vec![
            scope.gt("age", 18),
            scope.is_not_null("email"),
        ]),
    ]);
    scope.build_select("users", filter)
});
// Arena memory freed, sql String is owned

// ❌ Bad: Many small heap allocations
let filter = Filter::and(vec![
    Filter::Equals("active".into(), FilterValue::Bool(true)),
    Filter::or(vec![
        Filter::Gt("age".into(), FilterValue::Int(18)),
        Filter::IsNotNull("email".into()),
    ]),
]);
```

### Use Lazy Parsing

For large schemas where not all data is needed:

```rust
use prax_query::mem_optimize::lazy::LazySchema;

// ✅ Good: Lazy schema - only parses what you access
let schema = LazySchema::from_json(large_json)?;

// Table names available immediately (no parsing)
for name in schema.table_names() {
    if name == "users" {
        // Only now parse the users table
        let table = schema.get_table(name)?;
        for col in table.columns() {
            // Columns parsed on first access
        }
    }
}

// ❌ Bad: Parse everything eagerly
let schema: DatabaseSchema = serde_json::from_str(large_json)?;
// All tables and columns parsed upfront
```

### Avoid Unnecessary Allocations

```rust
// ✅ Good: Use references and slices
fn process(data: &[u8]) -> &str { ... }

// ✅ Good: Use Cow for conditional ownership
use std::borrow::Cow;

fn normalize(s: &str) -> Cow<'_, str> {
    if needs_normalization(s) {
        Cow::Owned(s.to_lowercase())
    } else {
        Cow::Borrowed(s)
    }
}

// ✅ Good: Reuse buffers
let mut buf = String::with_capacity(1024);
for item in items {
    buf.clear();
    write!(&mut buf, "{}", item)?;
    process(&buf);
}

// ❌ Bad: Allocate in a loop
for item in items {
    let s = format!("{}", item); // Allocation each iteration
    process(&s);
}
```

### Use SmallVec for Small Collections

```rust
use smallvec::SmallVec;

// ✅ Good: Inline storage for common case
let columns: SmallVec<[&str; 8]> = SmallVec::new();
// No heap allocation until > 8 elements

// ❌ Bad: Always heap allocate
let columns: Vec<&str> = Vec::new();
```

## Query Performance

### Batch Operations

```rust
// ✅ Good: Single multi-row INSERT
let mut builder = SqlBuilder::postgres();
builder.push("INSERT INTO events (user_id, type) VALUES ");
for (i, event) in events.iter().enumerate() {
    if i > 0 { builder.push(", "); }
    builder.push("(");
    builder.push_param(FilterValue::Int(event.user_id));
    builder.push(", ");
    builder.push_param(FilterValue::String(event.event_type.clone()));
    builder.push(")");
}

// ❌ Bad: Multiple INSERT statements
for event in events {
    execute("INSERT INTO events (user_id, type) VALUES ($1, $2)", &[...]).await?;
}
```

### Use Prepared Statements

```rust
// ✅ Good: Prepare once, execute many
let stmt = client.prepare("SELECT * FROM users WHERE id = $1").await?;
for id in ids {
    let row = client.query_one(&stmt, &[&id]).await?;
}

// ❌ Bad: Re-parse query each time
for id in ids {
    let row = client.query_one("SELECT * FROM users WHERE id = $1", &[&id]).await?;
}
```

### Efficient IN Clauses

```rust
// ✅ Good: Use ANY with array parameter (PostgreSQL)
let sql = "SELECT * FROM users WHERE id = ANY($1)";
let ids: Vec<i64> = vec![1, 2, 3, 4, 5];
client.query(sql, &[&ids]).await?;

// ✅ Good: Generate IN clause efficiently
let placeholders = postgres_in_pattern(1, ids.len());
let sql = format!("SELECT * FROM users WHERE id IN ({})", placeholders);

// ❌ Bad: Dynamic SQL with many parameters
let sql = format!(
    "SELECT * FROM users WHERE id IN ({})",
    ids.iter().map(|_| "?").collect::<Vec<_>>().join(", ")
);
```

### Select Only Needed Columns

```rust
// ✅ Good: Select specific columns
let select = Select::fields(["id", "email", "name"]);

// ❌ Bad: Select all columns when only few needed
let select = Select::all(); // SELECT * includes unused columns
```

## Async Performance

### Use Concurrent Execution

```rust
use prax_query::async_optimize::ConcurrentExecutor;

// ✅ Good: Execute independent queries in parallel
let executor = ConcurrentExecutor::new(config);
let results = executor.execute_batch(vec![

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/prax](https://github.com/quinnjr/prax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
