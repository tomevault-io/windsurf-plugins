---
trigger: always_on
description: This rule provides comprehensive best practices for Rust development, covering code organization, common patterns, performance, security, testing, pitfalls, and tooling. It aims to guide developers in writing idiomatic, efficient, secure, and maintainable Rust code.
---

# Rust Essentials - Must-Have Basics

**Purpose:** Core quality requirements for reliable Rust code generation. Use this for basic code generation tasks, learning Rust fundamentals, or when you need clean, safe code that compiles without warnings.

**When to use:** AI code generation, code reviews, teaching Rust basics, or any situation requiring solid foundation patterns.

## Core Quality Requirements

**CRITICAL: All generated Rust code MUST:**
- Compile without warnings on stable Rust
- Pass `cargo clippy --deny warnings`
- Follow `rustfmt` formatting standards
- Use Rust 2021 edition features
- Include proper error handling (no production `unwrap()`)

## Error Handling Fundamentals

**Rule: Never use `unwrap()`, `expect()`, or `panic!()` in production code paths.**
Why: These cause immediate program termination, making your application unreliable. Always return `Result<T, E>` for operations that can fail, allowing callers to decide how to handle errors.

**Use Result<T, E> for all fallible operations:**
```rust
// ✅ GOOD
pub fn read_config(path: &Path) -> Result<Config, ConfigError> {
    let content = fs::read_to_string(path)?;
    toml::from_str(&content).map_err(ConfigError::ParseError)
}

// ❌ BAD - never use unwrap() in production
pub fn read_config(path: &Path) -> Config {
    let content = fs::read_to_string(path).unwrap();
    toml::from_str(&content).unwrap()
}
```

**Rule: Use doc test features to ensure examples remain accurate and demonstrate different scenarios.**
Why: Doc tests are automatically run by `cargo test`, ensuring examples never become outdated. Use different doc test attributes to show various use cases and error conditions.

**Run doc tests with:** `cargo test --doc` or `cargo test` (includes all tests)

**Doc test best practices:**
```rust
/// Parses configuration from various sources.
/// 
/// # Examples
/// 
/// Basic usage:
/// ```
/// let config = parse_config("app.toml")?;
/// assert!(config.port > 0);
/// # Ok::<(), ConfigError>(())
/// ```
/// 
/// This example doesn't run but shows the API:
/// ```no_run
/// let config = parse_config("/etc/myapp/config.toml")?;
/// deploy_with_config(config);
/// # Ok::<(), Box<dyn std::error::Error>>(())
/// ```
/// 
/// Demonstrating error handling:
/// ```should_panic
/// let config = parse_config("nonexistent.toml").unwrap();
/// ```
/// 
/// Hidden setup code (lines starting with #):
/// ```
/// # use std::fs;
/// # fs::write("test.toml", "port = 8080").unwrap();
/// let config = parse_config("test.toml")?;
/// assert_eq!(config.port, 8080);
/// # fs::remove_file("test.toml").unwrap();
/// # Ok::<(), ConfigError>(())
/// ```
pub fn parse_config(path: &str) -> Result<Config, ConfigError> {
    // Implementation
}

**Rule: Create specific error types instead of using generic errors.**
Why: Specific errors enable proper error handling by callers and provide better debugging information. Use `thiserror` to reduce boilerplate.

**Define custom error types:**
```rust
#[derive(Debug, thiserror::Error)]
pub enum ConfigError {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
    #[error("Parse error: {0}")]
    ParseError(#[from] toml::de::Error),
}
```

**Rule: Use the `?` operator to propagate errors up the call stack.**
Why: The `?` operator provides clean error propagation without nested match statements. It automatically converts errors using the `From` trait.

**Use the ? operator for error propagation:**
```rust
pub fn process_user_data(id: u32) -> Result<UserProfile, UserError> {
    let user = database::find_user(id)?;  // Propagates database errors
    let profile = build_profile(&user)?;  // Propagates profile errors
    Ok(profile)
}
```

## Documentation Standards

**Rule: Every public function, struct, and module must have rustdoc comments with working code examples.**
Why: Documentation is part of the API contract. Good docs prevent misuse, reduce support burden, and make your code maintainable. Code examples are automatically tested by `cargo test`, ensuring documentation stays accurate.

**Every public item needs rustdoc with examples:**
```rust
/// Represents a user in the system.
///
/// # Examples
/// 
/// Creating a valid user:
/// ```
/// let user = User::new("alice@example.com", "Alice Smith")?;
/// assert_eq!(user.email(), "alice@example.com");
/// assert_eq!(user.name(), "Alice Smith");
/// # Ok::<(), UserError>(())
/// ```
/// 
/// Handling invalid email:
/// ```should_panic
/// let user = User::new("invalid-email", "Alice Smith").unwrap();
/// ```
///
/// # Errors
/// Returns `UserError::InvalidEmail` if email format is invalid.
#[derive(Debug, Clone)]
pub struct User {
    email: String,
    name: String,
}

impl User {
    /// Creates a new user with validated email.
    /// 
    /// # Examples
    /// ```
    /// use my_crate::User;
    /// 
    /// let user = User::new("bob@example.com", "Bob Jones")?;
    /// assert!(user.email().contains("@"));
    /// # Ok::<(), my_crate::UserError>(())
    /// ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rustic-ai/codeprism](https://github.com/rustic-ai/codeprism) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
