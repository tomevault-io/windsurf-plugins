---
trigger: always_on
description: This project uses **Rust Edition 2024** and follows modern Rust best practices.
---


# Rust 2024 Best Practices

This project uses **Rust Edition 2024** and follows modern Rust best practices.

## Edition Configuration

All `Cargo.toml` files must specify edition 2024:

```toml
[package]
name = "crate-name"
version = "0.1.0"
edition = "2024"
```

## Code Style

### Use `rustfmt`

Always format code with rustfmt before committing:

```bash
cargo fmt --all
```

### Use `clippy`

Run clippy and fix all warnings:

```bash
cargo clippy --all-features --all-targets -- -D warnings
```

### Naming Conventions

```rust
// ✅ Good
struct UserAccount { }           // Types: UpperCamelCase
trait Validate { }               // Traits: UpperCamelCase
enum HttpStatus { }              // Enums: UpperCamelCase
fn create_user() { }             // Functions: snake_case
const MAX_SIZE: usize = 100;     // Constants: SCREAMING_SNAKE_CASE
static GLOBAL_CONFIG: &str = ""; // Statics: SCREAMING_SNAKE_CASE
let user_name = "Alice";         // Variables: snake_case
mod http_client;                 // Modules: snake_case

// ❌ Bad
struct user_account { }          // Wrong case
fn CreateUser() { }              // Wrong case
const maxSize: usize = 100;      // Wrong case
```

## Error Handling

### Use `Result` and `?` Operator

```rust
// ✅ Good: Use Result and ? operator
pub fn parse_config(path: &str) -> Result<Config, Error> {
    let contents = std::fs::read_to_string(path)?;
    let config: Config = serde_json::from_str(&contents)?;
    validate_config(&config)?;
    Ok(config)
}

// ❌ Bad: Using unwrap in library code
pub fn parse_config(path: &str) -> Config {
    let contents = std::fs::read_to_string(path).unwrap(); // Don't panic!
    serde_json::from_str(&contents).unwrap()
}
```

### Use `thiserror` for Error Types

```rust
use thiserror::Error;

#[derive(Debug, Error)]
pub enum CacheError {
    #[error("Redis error: {0}")]
    Redis(#[from] redis::RedisError),

    #[error("Serialization error: {0}")]
    Serialization(String),

    #[error("Cache key not found: {0}")]
    NotFound(String),
}
```

### Never Use `unwrap()` or `expect()` in Library Code

```rust
// ✅ Good: Propagate errors
pub fn get_user(id: i32) -> Result<User, Error> {
    database.query("SELECT * FROM users WHERE id = ?", id)?
        .ok_or_else(|| Error::NotFound(format!("User {} not found", id)))
}

// ❌ Bad: Panicking in library code
pub fn get_user(id: i32) -> User {
    database.query("SELECT * FROM users WHERE id = ?", id)
        .unwrap()
        .expect("User not found") // Never use in library code!
}

// ✅ Acceptable: In application code, example code, or tests
#[tokio::main]
async fn main() {
    let app = Application::create(AppModule);
    app.listen(3000).await.expect("Failed to start server");
}
```

## Async/Await

### Use `async-trait` for Async Traits

```rust
use async_trait::async_trait;

#[async_trait]
pub trait Repository: Send + Sync {
    async fn find_by_id(&self, id: i32) -> Result<User, Error>;
    async fn save(&self, user: &User) -> Result<(), Error>;
}
```

### Prefer `tokio` Runtime

```rust
// ✅ Good: Use tokio for async runtime
#[tokio::main]
async fn main() -> Result<(), Error> {
    // Async code
    Ok(())
}

// For tests
#[tokio::test]
async fn test_async_function() {
    let result = async_function().await;
    assert!(result.is_ok());
}
```

### Use `.await` Properly

```rust
// ✅ Good: Await futures properly
let user = fetch_user(id).await?;
let posts = fetch_posts(user.id).await?;

// ✅ Good: Concurrent execution with join!
let (user, posts) = tokio::join!(
    fetch_user(id),
    fetch_posts(user_id)
);

// ❌ Bad: Blocking in async code
async fn bad_example() {
    std::thread::sleep(Duration::from_secs(1)); // Blocks entire thread!
}

// ✅ Good: Async sleep
async fn good_example() {
    tokio::time::sleep(Duration::from_secs(1)).await;
}
```

## Ownership and Borrowing

### Follow Ownership Rules

```rust
// ✅ Good: Clear ownership
pub struct User {
    pub id: i32,
    pub name: String,
}

impl User {
    // Takes ownership
    pub fn new(name: String) -> Self {
        Self { id: 0, name }
    }

    // Borrows immutably
    pub fn display(&self) {
        println!("{}", self.name);
    }

    // Borrows mutably
    pub fn update_name(&mut self, name: String) {
        self.name = name;
    }

    // Consumes self
    pub fn into_dto(self) -> UserDto {
        UserDto { name: self.name }
    }
}
```

### Use References Appropriately

```rust
// ✅ Good: Take references for read-only access
pub fn validate_email(email: &str) -> bool {
    email.contains('@')
}

// ❌ Bad: Unnecessary ownership
pub fn validate_email(email: String) -> bool {
    email.contains('@')
}

// ✅ Good: Return owned data
pub fn format_name(first: &str, last: &str) -> String {
    format!("{} {}", first, last)
}
```

### Prefer `&str` Over `&String`

```rust
// ✅ Good: Use &str for string parameters
pub fn process_text(text: &str) -> String {
    text.to_uppercase()
}

// ❌ Bad: Unnecessarily specific
pub fn process_text(text: &String) -> String {
    text.to_uppercase()
}
```

## Type Safety

### Use Newtypes for Type Safety

```rust
// ✅ Good: Type-safe wrappers
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub struct UserId(pub i32);

#[derive(Debug, Clone, Copy, PartialEq, Eq)]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/armature](https://github.com/quinnjr/armature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
