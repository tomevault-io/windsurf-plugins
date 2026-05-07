---
trigger: always_on
description: This project prioritizes **asynchronous and parallel execution** over synchronous code. All I/O operations, database queries, and potentially blocking operations must be async.
---

# Async/Parallel-First Design

This project prioritizes **asynchronous and parallel execution** over synchronous code. All I/O operations, database queries, and potentially blocking operations must be async.

## Core Principles

### 1. Default to Async

Every function that performs I/O should be `async`:

```rust
// ✅ Good: Async by default
pub async fn find_user(id: i64) -> Result<User> {
    let row = client.query_one("SELECT * FROM users WHERE id = $1", &[&id]).await?;
    Ok(User::from_row(row))
}

// ❌ Bad: Synchronous I/O
pub fn find_user(id: i64) -> Result<User> {
    let row = client.query_one("SELECT * FROM users WHERE id = $1", &[&id])?;
    Ok(User::from_row(row))
}
```

### 2. Parallel by Default

When multiple independent operations exist, execute them in parallel:

```rust
// ✅ Good: Parallel execution
let (users, posts, comments) = tokio::try_join!(
    client.user().find_many().exec(),
    client.post().find_many().exec(),
    client.comment().find_many().exec(),
)?;

// ❌ Bad: Sequential when parallel is possible
let users = client.user().find_many().exec().await?;
let posts = client.post().find_many().exec().await?;
let comments = client.comment().find_many().exec().await?;
```

### 3. Use `tokio::spawn` for CPU-bound Work

Offload CPU-intensive tasks to avoid blocking the async runtime:

```rust
// ✅ Good: Spawn blocking work
let parsed = tokio::task::spawn_blocking(move || {
    expensive_parsing_operation(&data)
}).await?;

// ❌ Bad: Blocking in async context
let parsed = expensive_parsing_operation(&data); // blocks the runtime
```

### 4. Prefer `tokio::select!` for Racing

When you need the first result from multiple futures:

```rust
// ✅ Good: Race with select
tokio::select! {
    result = primary_db.query(&sql) => handle_result(result),
    result = replica_db.query(&sql) => handle_result(result),
    _ = tokio::time::sleep(timeout) => return Err(Error::Timeout),
}
```

## Required Patterns

### Connection Pools

Always use async connection pools:

```rust
// Use deadpool-postgres or bb8
use deadpool_postgres::{Config, Pool, Runtime};

pub struct DatabasePool {
    pool: Pool,
}

impl DatabasePool {
    pub async fn get(&self) -> Result<PooledConnection> {
        self.pool.get().await.map_err(Into::into)
    }
}
```

### Streaming Results

For large result sets, use async streams:

```rust
use futures::Stream;
use tokio_stream::StreamExt;

pub fn find_all(&self) -> impl Stream<Item = Result<User>> {
    // Return a stream instead of Vec for memory efficiency
    async_stream::try_stream! {
        let mut rows = client.query_raw(&sql, &[]).await?;
        while let Some(row) = rows.next().await {
            yield User::from_row(row?);
        }
    }
}
```

### Batch Operations

Batch multiple queries for efficiency:

```rust
// ✅ Good: Batch inserts
pub async fn create_many(users: Vec<CreateUser>) -> Result<Vec<User>> {
    let futures: Vec<_> = users
        .into_iter()
        .map(|u| self.create(u))
        .collect();

    futures::future::try_join_all(futures).await
}
```

### Timeouts

Always include timeouts for async operations:

```rust
use tokio::time::{timeout, Duration};

pub async fn query_with_timeout(&self, sql: &str) -> Result<Vec<Row>> {
    timeout(Duration::from_secs(30), self.client.query(sql, &[]))
        .await
        .map_err(|_| Error::Timeout)?
        .map_err(Into::into)
}
```

## Trait Definitions

All database traits must be async:

```rust
// ✅ Good: Async trait (Rust 2024 supports this natively)
pub trait Repository {
    async fn find(&self, id: i64) -> Result<Option<Model>>;
    async fn find_many(&self, filter: Filter) -> Result<Vec<Model>>;
    async fn create(&self, data: CreateInput) -> Result<Model>;
    async fn update(&self, id: i64, data: UpdateInput) -> Result<Model>;
    async fn delete(&self, id: i64) -> Result<()>;
}

// ❌ Bad: Sync trait
pub trait Repository {
    fn find(&self, id: i64) -> Result<Option<Model>>;
}
```

## Concurrency Primitives

### Use `tokio::sync` Not `std::sync`

```rust
// ✅ Good: Tokio's async-aware primitives
use tokio::sync::{RwLock, Mutex, Semaphore, broadcast, mpsc};

// ❌ Bad: std sync primitives block the runtime
use std::sync::{RwLock, Mutex};
```

### Prefer `RwLock` Over `Mutex`

When reads dominate writes:

```rust
// ✅ Good: RwLock for read-heavy cache
let cache: Arc<RwLock<HashMap<K, V>>> = Arc::new(RwLock::new(HashMap::new()));

// Read path (many concurrent readers)
let value = cache.read().await.get(&key).cloned();

// Write path (exclusive access)
cache.write().await.insert(key, value);
```

## Error Handling in Async

### Propagate Errors with `?`

```rust
pub async fn complex_operation(&self) -> Result<Output> {
    let a = self.step_a().await?;
    let b = self.step_b(&a).await?;
    let c = self.step_c(&b).await?;
    Ok(c)
}
```

### Use `try_join!` for Parallel Error Handling

```rust
// All succeed or first error is returned
let (a, b, c) = tokio::try_join!(
    self.fetch_a(),
    self.fetch_b(),
    self.fetch_c(),
)?;
```

## Cancellation Safety

Document cancellation behavior for all public async functions:

```rust
/// Executes a database query.
///

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/prax](https://github.com/quinnjr/prax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
