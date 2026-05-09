---
trigger: always_on
description: Atomic is a mathematically sound distributed version control system built in Rust, emphasizing performance, correctness, and maintainability through sophisticated architectural patterns and best practices.
---

# AGENTS.md - Atomic Development Best Practices & Architecture Guide

## Project Overview

Atomic is a mathematically sound distributed version control system built in Rust, emphasizing performance, correctness, and maintainability through sophisticated architectural patterns and best practices.

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Configuration-Driven Design](#configuration-driven-design)
3. [Factory Pattern Implementation](#factory-pattern-implementation)
4. [Singleton Pattern Usage](#singleton-pattern-usage)
5. [DRY Principles with Macros](#dry-principles-with-macros)
6. [Error Handling Strategy](#error-handling-strategy)
7. [Code Organization Patterns](#code-organization-patterns)
8. [Development Guidelines](#development-guidelines)
9. [Testing Strategy](#testing-strategy)
10. [Performance Considerations](#performance-considerations)
11. [Environment Variable Detection Patterns](#environment-variable-detection-patterns)
12. [CLI Integration Patterns](#cli-integration-patterns)
13. [HTTP API Protocol Alignment](#http-api-protocol-alignment)
13. [HTTP API Protocol Alignment](#http-api-protocol-alignment)

## Architecture Overview

### Modular Crate Structure

The project follows a clean separation of concerns through focused crates:

```
atomic/                     # CLI application & command handlers
├── atomic-macros/          # Procedural macros for code generation
├── atomic-config/          # Configuration management system
├── atomic-identity/        # User identity & credential management
├── atomic-interaction/     # User interface & interaction patterns
├── atomic-remote/          # Remote repository operations
├── atomic-repository/      # Repository management
├── libatomic/             # Core VCS engine & algorithms
└── contrib/               # Additional resources
```

**Key Architectural Principles:**
- **Single Responsibility**: Each crate has one clear purpose
- **Dependency Inversion**: Core library (`libatomic`) has minimal dependencies
- **Interface Segregation**: Small, focused trait interfaces
- **Composition over Inheritance**: Struct composition with traits

### Database-Centric Architecture

Uses Sanakirja as the storage backend with macro-generated database operations:

```rust
// Example of database table generation
#[table("channel_changes")]
pub struct ChannelChanges {
    channel: ChannelRef,
    change: ChangeId,
}
```

## Configuration-Driven Design

### Hierarchical Configuration System

Atomic implements a sophisticated configuration hierarchy following the principle of **configuration over code**:

```rust
// Global configuration precedence:
// 1. Environment variables
// 2. Local repository config (.atomic/config.toml)
// 3. User-specific config (~/.config/atomic/config.toml)
// 4. System defaults

#[derive(Debug, Serialize, Deserialize, Default)]
pub struct Global {
    pub author: Author,
    pub unrecord_changes: Option<usize>,
    pub colors: Option<Choice>,
    pub pager: Option<Choice>,
    pub template: Option<Templates>,
    pub ignore_kinds: Option<HashMap<String, Vec<String>>>,
}
```

### Configuration Best Practices

1. **Serde Integration**: All configuration structs use serde for serialization
2. **Optional Fields**: Use `Option<T>` for optional configuration values
3. **Default Implementations**: Provide sensible defaults via `Default` trait
4. **Validation**: Validate configuration during deserialization
5. **Backward Compatibility**: Use serde aliases for renamed fields

```rust
#[derive(Debug, Clone, Serialize, Deserialize, PartialEq, Eq)]
pub struct Author {
    #[serde(alias = "name", default, skip_serializing_if = "String::is_empty")]
    pub username: String,
    #[serde(alias = "full_name", default, skip_serializing_if = "String::is_empty")]
    pub display_name: String,
}
```

### Remote Configuration Factory

The remote configuration system demonstrates the factory pattern:

```rust
#[derive(Debug, Serialize, Deserialize)]
#[serde(untagged)]
pub enum RemoteConfig {
    Ssh { name: String, ssh: String },
    Http { 
        name: String, 
        http: String,
        #[serde(default)]
        headers: HashMap<String, RemoteHttpHeader>,
    },
}

impl RemoteConfig {
    pub fn name(&self) -> &str {
        match self {
            RemoteConfig::Ssh { name, .. } => name,
            RemoteConfig::Http { name, .. } => name,
        }
    }
}
```

## Factory Pattern Implementation

### Identity Factory Pattern

The identity system uses a factory-like pattern for creating complete user identities:

```rust
impl Complete {
    /// Factory method for creating new identities
    pub fn new(
        name: String,
        config: Config,
        public_key: PublicKey,
        credentials: Option<Credentials>,
    ) -> Self {
        if name.is_empty() {
            panic!("Identity name cannot be empty!");
        }
        
        Self {
            name,
            config,
            public_key,
            credentials,
            last_modified: chrono::offset::Utc::now(),
        }
    }

    /// Factory method for default identity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atomicdotdev/atomic-pijul](https://github.com/atomicdotdev/atomic-pijul) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
