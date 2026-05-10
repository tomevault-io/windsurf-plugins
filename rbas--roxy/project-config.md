---
trigger: always_on
description: Roxy is a local development proxy tool written in Rust that enables developers to run multiple projects with custom `.roxy` domains and automatic HTTPS support. Think Laravel Valet, but written in Rust.
---

# Roxy - Claude Development Guide

## Project Overview

Roxy is a local development proxy tool written in Rust that enables developers to run multiple projects with custom `.roxy` domains and automatic HTTPS support. Think Laravel Valet, but written in Rust.

**Philosophy**: Pragmatic, idiomatic Rust. Build the simplest thing that works. YAGNI principles apply throughout.

## Rust Skills (AI)

This repo vendors `rust-skills` (skills-only, no MCP) at `.rust-skills/skills/`.

Local setup links those skills into:
- `~/.codex/skills/` (Codex)
- `~/.claude/skills/` (Claude Code)

When working on Rust tasks, prefer:
- Invoke `rust-router` first for Rust questions/errors/design.
- Then follow the routed skill (`m01-*`..`m07-*`, `m09-*`..`m15-*`, `domain-*`).
- Use `unsafe-checker` for any unsafe/FFI review.


## Core Principles

### 1. Idiomatic Rust

- Use standard library types and patterns
- Embrace `Result<T, E>` for error handling - no panics in library code
- Use `Option<T>` appropriately - avoid unnecessary unwrapping
- Prefer iterators over loops where it improves clarity
- Use enums for state and behavior variants
- Implement standard traits (`Display`, `Debug`, `From`, etc.) where appropriate
- Follow Rust naming conventions (snake_case for functions/variables, PascalCase for types)

### 2. YAGNI (You Aren't Gonna Need It)

- **Don't build features that aren't in REQUIREMENTS.md**
- Don't create abstractions until you need them in multiple places
- Don't add configuration options until someone asks for them
- Don't optimize until there's a proven performance issue
- Don't add plugins, hooks, or extensibility mechanisms in v1
- Start with simple implementations - refactor when complexity demands it

### 3. Error Handling

- Use `anyhow::Result` for application-level errors (CLI commands)
- Use custom error types (with `thiserror`) only when you need to match on error variants
- Provide helpful error messages - include context about what failed and why
- Never use `.unwrap()` or `.expect()` in production code paths
- Use `?` operator liberally for clean error propagation

### 4. Architecture Guidelines

#### Keep It Simple

- Start with a single binary with multiple modules
- Don't split into multiple crates unless there's a clear benefit
- Avoid over-abstraction - traits should solve real problems, not theoretical ones
- Direct implementation beats clever generics

#### Module Structure

```
roxy/
├── src/
│   ├── main.rs           # CLI entry point, argument parsing
│   ├── cli/              # CLI commands implementation
│   │   ├── mod.rs
│   │   ├── install.rs
│   │   ├── register.rs
│   │   ├── unregister.rs
│   │   └── list.rs
│   ├── daemon/           # HTTP server and proxy logic
│   │   ├── mod.rs
│   │   ├── server.rs
│   │   └── router.rs
│   ├── dns/              # DNS configuration management
│   │   └── mod.rs
│   ├── certs/            # Certificate generation and management
│   │   └── mod.rs
│   ├── config/           # Configuration storage and loading
│   │   └── mod.rs
│   └── lib.rs            # Library root (shared types and utilities)
```

#### Dependencies Philosophy

- **Minimize dependencies** - each new crate is a maintenance burden
- Prefer well-maintained, popular crates over niche ones
- Read the code of small crates before adding them
- Don't add a dependency to save 20 lines of code

### 5. Domain-Driven Design (Lightweight)

Use DDD concepts to make the code expressive and clear, but avoid DDD ceremony and over-engineering.

#### Value Objects

Wrap primitives in meaningful types to prevent mistakes and make the domain explicit:

```rust
// Value objects - immutable, validated on construction
pub struct DomainName(String);
pub struct Port(u16);

impl DomainName {
    pub fn new(name: impl Into<String>) -> Result<Self> {
        let name = name.into();
        if !name.ends_with(".local") {
            return Err(anyhow!("Domain must end with .local"));
        }
        if name.len() < 7 {  // Minimum: "a.local"
            return Err(anyhow!("Domain name too short"));
        }
        Ok(Self(name))
    }

    pub fn as_str(&self) -> &str {
        &self.0
    }
}

impl Display for DomainName {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{}", self.0)
    }
}
```

**When to use Value Objects:**

- Primitives with validation rules (DomainName, Port)
- Concepts that are compared by value (Certificate, FilePath)
- Types that should be immutable

**When NOT to use Value Objects:**

- Don't wrap primitives that have no validation or domain meaning
- Don't create value objects just to have them

#### Entities

Types with identity that can change over time:

```rust
// Entity - has identity (domain name), mutable state
pub struct DomainRegistration {
    domain: DomainName,
    target: Target,
    https_enabled: bool,
    created_at: SystemTime,
}

impl DomainRegistration {
    pub fn new(domain: DomainName, target: Target) -> Self {
        Self {
            domain,
            target,
            https_enabled: false,
            created_at: SystemTime::now(),
        }
    }


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rbas/roxy](https://github.com/rbas/roxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
