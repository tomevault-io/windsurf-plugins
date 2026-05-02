---
trigger: always_on
description: Rust development: ownership, lifetimes, error handling, async patterns, and cargo workflows
---


# Rust Development Patterns

Idiomatic Rust patterns focusing on ownership, safety, and performance.

## Rust Workflow

Before changing Rust code:

```text
1. Read `Cargo.toml` and current crate structure first
2. Check the edition and existing dependency set before recommending new patterns
3. For new crates, use `cargo new` or `cargo init`; do not hand-create `Cargo.toml`
4. For new dependencies or version-sensitive work, verify current versions with the actual current date
```

### CLI-First Rust Development

Prefer standard Cargo workflows:
```bash
# Project creation (NEVER manually create Cargo.toml)
cargo new my_project
cargo new --lib my_library
cargo init  # In existing directory

# Add dependencies (NEVER manually edit Cargo.toml for deps)
cargo add tokio --features full
cargo add serde --features derive
cargo add thiserror
cargo add anyhow

# Development dependencies
cargo add --dev tokio-test
cargo add --dev mockall

# Build and verify
cargo build
cargo check  # Faster than build, just checks
cargo clippy  # Linting
cargo fmt  # Format code

# Test
cargo test
cargo test -- --nocapture  # See println output
```

### Post-Edit Verification

After meaningful Rust changes, run the smallest useful check for the task:

```bash
cargo check
cargo test
cargo fmt --check
```

Add `cargo clippy` when the change is substantial, safety-sensitive, or lint-heavy.

### Common Rust Syntax Traps (Avoid These!)

```rust
// WRONG: Using unwrap in production code
let value = some_option.unwrap();  // Panics on None!
let data = result.unwrap();  // Panics on Err!

// CORRECT: Handle errors properly
let value = some_option.ok_or(MyError::NotFound)?;
let data = result.map_err(|e| MyError::from(e))?;

// WRONG: Borrowing across await points
async fn bad_example(data: &mut Data) {
    let reference = &data.field;
    async_operation().await;  // reference held across await!
    use_reference(reference);
}

// CORRECT: Clone or restructure
async fn good_example(data: &mut Data) {
    let value = data.field.clone();
    async_operation().await;
    use_value(value);
}

// WRONG: Missing Send bound for async traits
trait MyAsyncTrait {
    async fn do_work(&self);  // Won't compile in multi-threaded!
}

// CORRECT: Add Send bound when needed
trait MyAsyncTrait: Send + Sync {
    fn do_work(&self) -> impl Future<Output = ()> + Send;
}

// WRONG: String vs &str confusion
fn greet(name: String) { }  // Takes ownership unnecessarily
greet("hello".to_string());  // Wasteful allocation

// CORRECT: Accept borrowed when possible
fn greet(name: &str) { }  // Borrows, no allocation needed
greet("hello");  // Works directly with string literal
```

### Cargo.toml Best Practices

```toml
[package]
name = "myproject"
version = "0.1.0"
edition = "2021"  # Always specify edition
rust-version = "1.75"  # Minimum Rust version

[dependencies]
# Pin to semver-compatible range
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }

[dev-dependencies]
tokio-test = "0.4"

# Optimize release builds
[profile.release]
lto = true
codegen-units = 1
```

---

## Ownership & Borrowing

### Core Principles
1. Each value has exactly one owner
2. When the owner goes out of scope, the value is dropped
3. References must always be valid
4. Either one mutable reference OR any number of immutable references

### Borrowing Patterns
```rust
// Immutable borrow - read only
fn print_length(s: &str) {
    println!("Length: {}", s.len());
}

// Mutable borrow - can modify
fn append_suffix(s: &mut String) {
    s.push_str("_suffix");
}

// Taking ownership - consumes the value
fn consume_string(s: String) {
    println!("Consumed: {}", s);
    // s is dropped here
}

// Returning ownership
fn create_string() -> String {
    String::from("created")
}
```

### When to Use What
- `&T`: Reading data, most function parameters
- `&mut T`: Modifying data in place
- `T`: Taking ownership, returning from functions, storing in structs

---

## Lifetimes

### Basic Lifetime Annotations
```rust
// The returned reference lives as long as the shortest input lifetime
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

// Struct with references
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
    
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention: {}", announcement);
        self.part
    }
}
```

### Lifetime Elision Rules
The compiler infers lifetimes when:
1. Each reference parameter gets its own lifetime
2. If exactly one input lifetime, it's assigned to all outputs
3. If `&self` or `&mut self`, that lifetime is assigned to outputs

```rust
// These are equivalent:
fn first_word(s: &str) -> &str { ... }
fn first_word<'a>(s: &'a str) -> &'a str { ... }
```

---

## Error Handling

### Result and Option
```rust
use std::fs::File;
use std::io::{self, Read};

// Using Result
fn read_file(path: &str) -> Result<String, io::Error> {
    let mut file = File::open(path)?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

// Using Option
fn find_user(id: u64) -> Option<User> {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m2-cursor-rules](https://github.com/madebyaris/advance-minimax-m2-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
