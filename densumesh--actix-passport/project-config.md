---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

actix-passport is a flexible authentication framework for actix-web applications in Rust. It provides multiple authentication methods including password-based auth, OAuth 2.0, JWT tokens, and session-based authentication.

## Recent Development Context (Updated Dec 2024)

### Pre-Built Database Store Implementation
We have successfully implemented the foundation for pre-built database stores to drastically reduce integration complexity:

**Completed:**
- ✅ Feature flag system (`postgres`, `mysql`, `sqlite`, `mongodb`, `redis`)
- ✅ Comprehensive store infrastructure with shared utilities
- ✅ Production-ready PostgreSQL store using SQLx
- ✅ Builder convenience methods (`ActixPassportBuilder::with_postgres_store()`)
- ✅ Automatic migrations and connection pooling
- ✅ Developer experience: 500+ lines → 5 lines integration

**Current Database Integration Options:**
```rust
// In-memory (development/testing)
let auth = ActixPassportBuilder::with_in_memory_store()
    .enable_password_auth()
    .build();

// PostgreSQL (production-ready)
let auth = ActixPassportBuilder::with_postgres_store("postgres://...")
    .await?
    .enable_password_auth()
    .build();
```

### SQLx vs Diesel Decision
**Current Implementation:** SQLx (chosen for simpler async, better JSON handling)

**Known Issue:** Diesel users face significant pain points:
- Dependency conflicts (dual database drivers)
- Connection pool duplication
- Transaction isolation issues
- Migration system conflicts
- Type system fragmentation

**Recommended Next Steps:** Add Diesel support alongside SQLx to provide choice:
```rust
// Feature flags for both ORMs
postgres-sqlx = ["dep:sqlx", ...]
postgres-diesel = ["dep:diesel", ...]

// Builder methods for both
ActixPassportBuilder::with_postgres_sqlx_store("...")
ActixPassportBuilder::with_postgres_diesel_store(existing_diesel_pool)
```

This would allow Diesel users to integrate with their existing infrastructure without forcing SQLx adoption.

## Development Commands

### Build and Test
```bash
cargo build                         # Build the project (core features only)
cargo build --features postgres     # Build with PostgreSQL support
cargo test                          # Run all tests
cargo test --features postgres      # Test with PostgreSQL features
cargo clippy                        # Run linter (configured with strict rules)
```

### Examples
```bash
cd examples/basic_example && cargo run    # Run basic password auth example
```

## Core Architecture

### Framework Structure
The framework is built around several key components:

1. **ActixPassport** - Main framework object created via `ActixPassportBuilder`
2. **UserStore trait** - Pluggable user persistence (database-agnostic)
3. **Password authentication** - Built-in Argon2 password hashing
4. **OAuthProvider trait** - Extensible OAuth provider system
5. **Authentication middleware** - Session and JWT-based auth middleware

### Module Organization
- `src/user_store/` - Core UserStore trait and implementations
  - `src/user_store/stores/` - Pre-built store implementations
    - `src/user_store/stores/in_memory.rs` - In-memory store (development)
    - `src/user_store/stores/postgres.rs` - PostgreSQL store (SQLx-based)
    - `src/user_store/stores/common/` - Shared utilities for database stores
      - `sql_helpers.rs` - Common SQL patterns and queries
      - `json_helpers.rs` - JSON serialization utilities
      - `error_mapping.rs` - Database error → AuthError conversion
      - `migrations.rs` - Migration system foundation
- `src/builder.rs` - Builder pattern with convenience methods for stores
- `src/types.rs` - Core types (`AuthUser`, `AuthResult`)
- `src/middleware/` - Authentication middleware implementations
- `src/routes/` - Built-in authentication route handlers
- `src/password/` - Password hashing and authentication (feature gated)
- `src/oauth/` - OAuth 2.0 providers and service (feature gated)

### Key Traits to Implement
When extending the framework, you'll typically implement:
- `UserStore` for custom database backends
- `OAuthProvider` for custom OAuth providers

### Builder Pattern Usage
The framework uses a builder pattern for configuration:

**Manual UserStore (custom implementations):**
```rust
let auth = ActixPassportBuilder::new(custom_user_store)
    .enable_password_auth()  // Uses Argon2 hashing internally
    .with_oauth(provider)
    .build();
```

**Pre-built Stores (recommended):**
```rust
// In-memory (development)
let auth = ActixPassportBuilder::with_in_memory_store()
    .enable_password_auth()
    .build();

// PostgreSQL (production)
let auth = ActixPassportBuilder::with_postgres_store("postgres://...")
    .await?
    .enable_password_auth()
    .build();
```

### Feature Flags
**Core Features:**
- `password` (default) - Username/password authentication
- `oauth` (default) - OAuth 2.0 providers  

**Database Store Features:**
- `postgres` - PostgreSQL store with SQLx (production-ready)
- `mysql` - MySQL store with SQLx (planned)
- `sqlite` - SQLite store with SQLx (planned)
- `mongodb` - MongoDB store (planned)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [densumesh/actix-passport](https://github.com/densumesh/actix-passport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
