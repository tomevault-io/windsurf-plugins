---
trigger: always_on
description: Guidelines for maintaining README.md documentation for the Prax ORM project
---


# README Guidelines

This document provides guidelines for maintaining the project README to ensure clear, accurate, and helpful documentation.

## README Structure

The README should follow this structure:

```markdown
# Prax

<badges and shields>

<brief description>

## Features
## Installation
## Quick Start
## Query Operations
## Architecture
## CLI
## Comparison
## Contributing
## License
## Acknowledgments
```

## When to Update

### Always Update When:
- ✅ Adding new public API methods
- ✅ Changing installation requirements
- ✅ Adding new features mentioned in examples
- ✅ Changing CLI commands
- ✅ Adding new database backend support
- ✅ Adding new framework integrations
- ✅ Changing minimum Rust version

### Consider Updating When:
- 🤔 Fixing bugs that affect documented behavior
- 🤔 Improving performance significantly
- 🤔 Adding new optional features

### Don't Update For:
- ❌ Internal refactoring
- ❌ Test changes
- ❌ Minor bug fixes
- ❌ Dependency updates (unless breaking)

## Section Guidelines

### Header & Badges

```markdown
# Prax

<p align="center">
  <strong>A next-generation, type-safe ORM for Rust</strong>
</p>

<p align="center">
  <a href="https://crates.io/crates/prax"><img src="https://img.shields.io/crates/v/prax.svg" alt="crates.io"></a>
  <a href="https://docs.rs/prax"><img src="https://docs.rs/prax/badge.svg" alt="docs.rs"></a>
  <a href="https://github.com/pegasusheavy/prax/actions"><img src="https://github.com/pegasusheavy/prax/workflows/CI/badge.svg" alt="CI"></a>
  <img src="https://img.shields.io/badge/rust-1.85%2B-blue.svg" alt="Rust 1.85+">
  <a href="#license"><img src="https://img.shields.io/badge/license-MIT%2FApache--2.0-blue.svg" alt="License"></a>
</p>
```

### Features Section

List features with emoji icons for scannability:

```markdown
## Features

- 🔒 **Type-Safe Queries** - Compile-time checked queries
- ⚡ **Async-First** - Built on Tokio
- 🎯 **Fluent API** - Intuitive query builder
- 🔗 **Relations** - Eager and lazy loading
- 📦 **Migrations** - Schema management
- 🛠️ **Code Generation** - Proc-macro models
- 🗄️ **Multi-Database** - PostgreSQL, MySQL, SQLite
- 🔌 **Framework Integration** - Armature, Axum, Actix-web
```

### Installation Section

Always show:
1. Basic installation
2. Feature flags for different backends
3. Minimum Rust version requirement

```markdown
## Installation

Add Prax to your `Cargo.toml`:

\`\`\`toml
[dependencies]
prax = "0.1"
\`\`\`

**Requires Rust 1.85+** (Edition 2024)

### Feature Flags

| Feature | Description |
|---------|-------------|
| `postgres` | PostgreSQL support (default) |
| `mysql` | MySQL support |
| `sqlite` | SQLite support |
| `runtime-tokio` | Tokio runtime (default) |
```

### Quick Start Section

Provide a complete, copy-pasteable example:

```markdown
## Quick Start

\`\`\`rust
use prax::prelude::*;

#[derive(Model)]
#[prax(table = "users")]
pub struct User {
    #[prax(id)]
    pub id: i32,
    pub email: String,
}

#[tokio::main]
async fn main() -> Result<(), prax::Error> {
    let client = PraxClient::new("postgresql://localhost/mydb").await?;

    let users = client.user().find_many().exec().await?;

    Ok(())
}
\`\`\`
```

### Code Examples

#### DO ✅

```rust
// Good: Complete, runnable example
use prax::prelude::*;

let users = client
    .user()
    .find_many()
    .where_(user::active::equals(true))
    .order_by(user::created_at::desc())
    .take(10)
    .exec()
    .await?;
```

#### DON'T ❌

```rust
// Bad: Incomplete, won't compile
let users = client.user().find_many()...
```

### API Documentation

When documenting query operations, use tables for clarity:

```markdown
## Query Operations

### Filtering

| Method | SQL Equivalent | Example |
|--------|---------------|---------|
| `equals(v)` | `= v` | `user::id::equals(1)` |
| `not_equals(v)` | `!= v` | `user::status::not_equals("banned")` |
| `contains(v)` | `LIKE %v%` | `user::name::contains("john")` |
| `gt(v)` | `> v` | `user::age::gt(18)` |
```

### Architecture Section

Keep the directory tree updated:

```markdown
## Architecture

\`\`\`
prax/
├── prax-core/           # Core types and traits
├── prax-schema/         # Schema parser
├── prax-codegen/        # Proc-macros
├── prax-query/          # Query builder
├── prax-postgres/       # PostgreSQL driver
├── prax-mysql/          # MySQL driver
├── prax-sqlite/         # SQLite driver
├── prax-migrate/        # Migrations
├── prax-cli/            # CLI tool
├── prax-armature/       # Armature integration
└── prax/                # Main crate
\`\`\`
```

### Comparison Table

Keep comparisons fair and up-to-date:

```markdown
## Comparison

| Feature | Prax | Diesel | SeaORM | SQLx |
|---------|------|--------|--------|------|
| Async | ✅ | ❌ | ✅ | ✅ |
| Type-Safe | ✅ | ✅ | ✅ | ✅ |
| Schema DSL | ✅ | ❌ | ❌ | ❌ |
| Migrations | ✅ | ✅ | ✅ | ✅ |
```

## Writing Style

### Tone
- Professional but approachable
- Confident but not arrogant
- Technical but accessible

### Formatting
- Use code blocks with language hints
- Use tables for structured data
- Use emoji sparingly for visual scanning
- Keep paragraphs short (2-3 sentences max)

### Links
- Link to detailed docs for complex topics
- Use relative links for repo files: `[CONTRIBUTING](./CONTRIBUTING.md)`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/prax](https://github.com/quinnjr/prax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
