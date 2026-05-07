---
trigger: always_on
description: This project uses `thiserror` for library errors and follows Rust error handling best practices.
---

# Error Handling Guidelines

This project uses `thiserror` for library errors and follows Rust error handling best practices.

## Error Types

### Use `thiserror` for Library Errors

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum QueryError {
    #[error("connection failed: {0}")]
    Connection(#[source] tokio_postgres::Error),

    #[error("query timeout after {0:?}")]
    Timeout(std::time::Duration),

    #[error("invalid filter: {field} - {message}")]
    InvalidFilter { field: String, message: String },

    #[error("record not found: {entity} with id {id}")]
    NotFound { entity: &'static str, id: i64 },

    #[error("constraint violation: {0}")]
    Constraint(String),
}
```

### Error Hierarchy

```rust
// Top-level error type
#[derive(Error, Debug)]
pub enum PraxError {
    #[error("query error: {0}")]
    Query(#[from] QueryError),

    #[error("schema error: {0}")]
    Schema(#[from] SchemaError),

    #[error("migration error: {0}")]
    Migration(#[from] MigrationError),

    #[error("connection error: {0}")]
    Connection(#[from] ConnectionError),
}

// Domain-specific errors
#[derive(Error, Debug)]
pub enum SchemaError {
    #[error("parse error at line {line}: {message}")]
    Parse { line: usize, message: String },

    #[error("invalid model: {0}")]
    InvalidModel(String),

    #[error("unknown type: {0}")]
    UnknownType(String),
}
```

## Error Propagation

### Use `?` Operator

```rust
// ✅ Good: Clean error propagation
pub async fn find_user(id: i64) -> Result<User, QueryError> {
    let conn = self.pool.get().await?;
    let row = conn.query_one(&self.sql, &[&id]).await?;
    let user = User::from_row(row)?;
    Ok(user)
}

// ❌ Bad: Explicit match everywhere
pub async fn find_user(id: i64) -> Result<User, QueryError> {
    let conn = match self.pool.get().await {
        Ok(c) => c,
        Err(e) => return Err(e.into()),
    };
    // ... more matches ...
}
```

### Add Context with `map_err`

```rust
use std::path::Path;

pub fn read_schema(path: &Path) -> Result<Schema, SchemaError> {
    let content = std::fs::read_to_string(path)
        .map_err(|e| SchemaError::Io {
            path: path.to_path_buf(),
            source: e,
        })?;

    parse_schema(&content)
        .map_err(|e| SchemaError::Parse {
            path: path.to_path_buf(),
            source: e,
        })
}
```

### Use `anyhow` for Context in Applications

```rust
// In CLI or application code (not library)
use anyhow::{Context, Result};

pub fn run_migration(path: &str) -> Result<()> {
    let schema = read_schema(path)
        .with_context(|| format!("failed to read schema from {}", path))?;

    let sql = generate_sql(&schema)
        .context("failed to generate SQL")?;

    execute_sql(&sql)
        .context("failed to execute migration")?;

    Ok(())
}
```

## Error Design Principles

### Make Errors Actionable

```rust
// ✅ Good: Error tells you what went wrong and how to fix it
#[error("field '{field}' requires type {expected}, got {actual}. Use @{expected} attribute or change the type.")]
InvalidFieldType {
    field: String,
    expected: &'static str,
    actual: String,
}

// ❌ Bad: Vague error
#[error("invalid field")]
InvalidField,
```

### Include Relevant Data

```rust
// ✅ Good: Error includes debugging information
#[error("query failed after {attempts} attempts (last error: {last_error})")]
RetryExhausted {
    attempts: u32,
    last_error: String,
    query: String, // Include the query for debugging
}

// ❌ Bad: No context
#[error("retry failed")]
RetryFailed,
```

### Preserve Error Chain

```rust
#[derive(Error, Debug)]
pub enum DatabaseError {
    // ✅ Good: Preserves source error
    #[error("connection pool exhausted")]
    PoolExhausted(#[source] deadpool::PoolError),

    // ✅ Good: #[from] for automatic conversion
    #[error("postgres error")]
    Postgres(#[from] tokio_postgres::Error),
}
```

## Handling Specific Error Cases

### Not Found vs Error

```rust
// Return Option for "not found" when that's a valid state
pub async fn find_by_id(id: i64) -> Result<Option<User>, QueryError> {
    match self.query_one(&sql, &[&id]).await {
        Ok(row) => Ok(Some(User::from_row(row)?)),
        Err(e) if e.is_no_rows() => Ok(None),
        Err(e) => Err(e.into()),
    }
}

// Return Error for "not found" when it indicates a problem
pub async fn get_by_id(id: i64) -> Result<User, QueryError> {
    self.find_by_id(id)
        .await?
        .ok_or(QueryError::NotFound { entity: "User", id })
}
```

### Validation Errors

```rust
#[derive(Error, Debug)]
pub enum ValidationError {
    #[error("validation failed")]
    Multiple(Vec<FieldError>),
}

#[derive(Debug)]
pub struct FieldError {
    pub field: String,
    pub message: String,
    pub code: &'static str,
}

impl ValidationError {
    pub fn single(field: impl Into<String>, message: impl Into<String>) -> Self {
        Self::Multiple(vec![FieldError {
            field: field.into(),
            message: message.into(),
            code: "invalid",
        }])
    }

    pub fn builder() -> ValidationErrorBuilder {
        ValidationErrorBuilder::new()
    }
}

// Usage
let mut errors = ValidationError::builder();
if email.is_empty() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/prax](https://github.com/quinnjr/prax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
