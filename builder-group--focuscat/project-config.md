---
trigger: always_on
description: Rust coding standards and style guidelines for Tauri applications
---


# Rust Style Guide

Rust coding standards and style guidelines for our Tauri applications. These guidelines ensure consistency, maintainability, and high code quality across Rust codebases.

## Core Principles

- **KISS (Keep It Simple, Stupid)** - Always choose the simplest, most maintainable solution
- **Type Safety First** - Always leverage Rust's type system to prevent errors at compile time
- **Less is More** - Always avoid unnecessary complexity, the best code is no code
- **Intent-Driven Comments** - Always use comments to explain intent and categorize code, never to restate what the code does

## Code Organization

### Module Structure
- Always organize code in a predictable and scalable way
- Always keep related code close together
- Always use clear, descriptive module names
- Always follow consistent patterns across the project

✅ Good:
```rust
src/
  features/
    activity_window/
      mod.rs
      types.rs
      repository.rs
      watcher.rs
      commands.rs
  common/
    db.rs
    time.rs
```

❌ Bad:
```rust
src/
  features/
    ActivityWindow/     # Wrong: PascalCase directory
    activity_window/
      Types.rs          # Wrong: PascalCase file
      repository.rs
      watcher.rs
```

### File Naming
- Always use `snake_case` for Rust files
- Always use descriptive, purpose-indicating names
- Always follow Rust community conventions

✅ Good:
```rust
user_service.rs
jwt_utils.rs
activity_repository.rs
```

❌ Bad:
```rust
UserService.rs      # Wrong: PascalCase
user-service.rs     # Wrong: kebab-case
USER_UTILS.rs       # Wrong: UPPER_SNAKE_CASE
```

## Comments and Documentation

### Comment Philosophy
- Always use comments to explain **intent** and **context**, not to restate code
- Always use comments to **categorize** and **section** code into logical blocks
- Always use comments to provide **context** that isn't obvious from the code
- Never restate what the code already makes clear
- Never add comments that become outdated quickly

✅ Good:
```rust
/// Upsert item (insert or update if exists).
pub async fn upsert(input: &UpsertInput) -> Result<i64, Error> {
    // Try to find existing item
    let existing: Option<i64> = query("SELECT id FROM items WHERE key = ?")
        .bind(&input.key)
        .fetch_optional(pool)
        .await?;

    // Update existing item
    if let Some(id) = existing {
        query("UPDATE items SET name = ? WHERE id = ?")
            .execute(pool)
            .await?;
        return Ok(id);
    }
    // Insert new item
    else {
        let id = query("INSERT INTO items ... RETURNING id")
            .fetch_one(pool)
            .await?;
        return Ok(id);
    }
}
```

❌ Bad:
```rust
// This function upserts an item
pub async fn upsert(input: &UpsertInput) -> Result<i64, Error> {
    // Get current timestamp
    let now = current_timestamp();

    // Query the database
    let existing = query("SELECT id...")
        .bind(&input.key)  // Bind key
        .fetch_optional(pool)  // Fetch optional
        .await?;  // Await

    // Check if existing exists
    if let Some(id) = existing {
        // Update the item
        query("UPDATE...").execute(pool).await?;
        // Return the id
        return Ok(id);
    }
}
```

### Section Dividers
- Always use `// MARK: -` style section dividers (Xcode convention)
- Always use section dividers to organize large files into logical sections
- Always place dividers before major sections, not between every function

✅ Good:
```rust
// MARK: - App Repository

pub struct AppRepository;

impl AppRepository {
    // ...
}

// MARK: - App Activity Repository

pub struct AppActivityRepository;
```

❌ Bad:
```rust
// App Repository
pub struct AppRepository;

// Implementation
impl AppRepository {
    // Upsert method
    pub async fn upsert(...) {
        // ...
    }
    // Get method
    pub async fn get(...) {
        // ...
    }
}
```

### Doc Comments
- Always use doc comments (`///`) for public APIs
- Always explain behavior, edge cases, and return values
- Always use doc comments for structs, enums, and public functions

✅ Good:
```rust
/// Insert new session.
/// Returns None if duration is zero or negative (entry skipped).
pub async fn insert(input: &UpsertInput) -> Result<Option<i64>, Error> {
    // Skip entries with zero or negative duration
    if input.end_time <= input.start_time {
        return Ok(None);
    }
    // ...
}
```

❌ Bad:
```rust
// Insert activity
pub async fn insert(input: &UpsertInput) -> Result<Option<i64>, Error> {
    // ...
}
```

## Return Statements

### Explicit Returns
- Always use explicit `return` keyword for return statements
- Always prefer clarity over implicit returns
- Always use `return` for early returns

✅ Good:
```rust
pub async fn upsert(input: &UpsertInput) -> Result<i64, Error> {
    if let Some(id) = existing {
        update_item(id).await?;
        return Ok(id);
    } else {
        let id = insert_item(input).await?;
        return Ok(id);
    }
}

fn validate(input: &str) -> Result<(), Error> {
    if input.is_empty() {
        return Err(Error::InvalidInput);
    }
    if input.len() > 100 {
        return Err(Error::InputTooLong);
    }
    Ok(())
}
```

❌ Bad:
```rust
pub async fn upsert(input: &UpsertInput) -> Result<i64, Error> {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [builder-group/focuscat](https://github.com/builder-group/focuscat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
