---
trigger: always_on
description: This project provides a public API for ORM operations. Follow these guidelines for consistent, ergonomic, and safe API design.
---

# API Design Guidelines

This project provides a public API for ORM operations. Follow these guidelines for consistent, ergonomic, and safe API design.

## Builder Pattern

### Use Builders for Complex Construction

```rust
// ✅ Good: Builder pattern for many options
let query = client
    .user()
    .find_many()
    .where_(user::status::equals("active"))
    .where_(user::age::gte(18))
    .order_by(user::created_at::desc())
    .take(10)
    .skip(20)
    .select(user::select!([id, email, name]))
    .exec()
    .await?;

// ✅ Good: Builder with required fields enforced by types
let config = DatabaseConfig::builder()
    .url("postgres://...")  // Required
    .max_connections(10)    // Optional
    .build()?;              // Validates required fields
```

### Return Self for Chaining

```rust
pub struct QueryBuilder {
    filter: Option<Filter>,
    order_by: Option<OrderBy>,
    take: Option<usize>,
}

impl QueryBuilder {
    // ✅ Good: Return Self for chaining
    pub fn where_(mut self, filter: Filter) -> Self {
        self.filter = Some(filter);
        self
    }

    pub fn order_by(mut self, order: OrderBy) -> Self {
        self.order_by = Some(order);
        self
    }

    pub fn take(mut self, n: usize) -> Self {
        self.take = Some(n);
        self
    }
}
```

### Use `#[must_use]` for Builders

```rust
#[must_use = "queries do nothing until .exec() is called"]
pub struct QueryBuilder { ... }

#[must_use = "this returns a new builder and does not modify self"]
pub fn where_(self, filter: Filter) -> Self { ... }
```

## Type-Safe APIs

### Use Enums Instead of Strings

```rust
// ✅ Good: Type-safe ordering
pub enum SortDirection {
    Asc,
    Desc,
}

pub fn order_by(column: &str, direction: SortDirection) -> OrderBy { ... }

// ❌ Bad: Stringly typed
pub fn order_by(column: &str, direction: &str) -> OrderBy { ... }
// Caller can pass "ascending", "ASC", "up", etc.
```

### Use Newtypes for Domain Concepts

```rust
// ✅ Good: Newtypes prevent mixing up arguments
#[derive(Debug, Clone, Copy)]
pub struct UserId(pub i64);

#[derive(Debug, Clone, Copy)]
pub struct PostId(pub i64);

pub fn get_posts_by_user(user_id: UserId) -> Vec<Post> { ... }

// ❌ Bad: Easy to mix up i64 arguments
pub fn get_posts_by_user(user_id: i64) -> Vec<Post> { ... }
// get_posts_by_user(post_id) compiles but is wrong!
```

### Use `Into` for Flexible Input

```rust
// ✅ Good: Accept multiple input types
pub fn where_<F: Into<Filter>>(self, filter: F) -> Self {
    self.filter = Some(filter.into());
    self
}

// Caller can use:
// .where_(Filter::Equals(...))
// .where_(user::id::equals(1))
// .where_("id = 1")  // if impl Into<Filter> for &str
```

## Error Handling in APIs

### Return Result for Fallible Operations

```rust
// ✅ Good: Result for operations that can fail
pub async fn exec(self) -> Result<Vec<User>, QueryError> { ... }

// ✅ Good: Option for "not found" scenarios
pub async fn find_unique(self) -> Result<Option<User>, QueryError> { ... }

// ❌ Bad: Panic on error
pub async fn exec(self) -> Vec<User> {
    self.try_exec().unwrap() // Don't panic in library code!
}
```

### Document Error Conditions

```rust
/// Execute the query and return results.
///
/// # Errors
///
/// Returns `QueryError::Connection` if the database is unreachable.
/// Returns `QueryError::Timeout` if the query exceeds the configured timeout.
/// Returns `QueryError::InvalidFilter` if the filter references unknown columns.
pub async fn exec(self) -> Result<Vec<User>, QueryError> { ... }
```

## Naming Conventions

### Methods

```rust
// Constructors: new, with_*, from_*
pub fn new() -> Self { ... }
pub fn with_capacity(n: usize) -> Self { ... }
pub fn from_row(row: Row) -> Self { ... }

// Queries: is_*, has_*, can_*
pub fn is_empty(&self) -> bool { ... }
pub fn has_filter(&self) -> bool { ... }

// Getters: no prefix, or get_* for clarity
pub fn len(&self) -> usize { ... }
pub fn get_column(&self, name: &str) -> Option<&Column> { ... }

// Setters: set_* or builder-style
pub fn set_filter(&mut self, filter: Filter) { ... }
pub fn filter(self, filter: Filter) -> Self { ... } // Builder

// Conversions: to_*, into_*, as_*
pub fn to_sql(&self) -> String { ... }           // Allocates
pub fn into_inner(self) -> T { ... }             // Consumes self
pub fn as_str(&self) -> &str { ... }             // Borrows
```

### Types

```rust
// Traits: Verb or -able/-ible
pub trait Execute { ... }
pub trait Filterable { ... }

// Builders: *Builder
pub struct QueryBuilder { ... }
pub struct ConfigBuilder { ... }

// Errors: *Error
pub enum QueryError { ... }
pub enum ParseError { ... }

// Results: Use type aliases
pub type QueryResult<T> = Result<T, QueryError>;
```

## Documentation

### Document Public Items

```rust
/// A type-safe query builder for database operations.
///
/// # Examples
///
/// ```rust
/// let users = client
///     .user()
///     .find_many()
///     .where_(user::active::equals(true))
///     .take(10)
///     .exec()
///     .await?;
/// ```
///
/// # Panics
///
/// This method never panics.
///
/// # Errors
///
/// Returns `QueryError` if the database query fails.
pub struct QueryBuilder { ... }
```

### Provide Examples in Docs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/prax](https://github.com/quinnjr/prax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
