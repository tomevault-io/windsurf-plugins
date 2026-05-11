---
trigger: always_on
description: Guidelines for creating new modules/crates in the Armature framework.
---


# Armature Module Development

Guidelines for creating new modules/crates in the Armature framework.

## Module Structure

Every new armature module should follow this structure:

```
armature-<name>/
├── Cargo.toml
├── src/
│   ├── lib.rs           # Public API exports
│   ├── config.rs        # Configuration types (if applicable)
│   ├── error.rs         # Module-specific error types
│   ├── traits.rs        # Core traits for the module
│   └── ...              # Implementation files
└── tests/
    └── integration.rs   # Integration tests
```

## Cargo.toml Template

```toml
[package]
name = "armature-<name>"
version.workspace = true
edition.workspace = true
rust-version.workspace = true
authors.workspace = true
license.workspace = true
repository.workspace = true
homepage.workspace = true
description = "Brief description of the module"
keywords = ["armature", "<relevant>", "<keywords>"]
categories = ["web-programming"]

[dependencies]
# Core dependencies - use workspace versions when available
tokio = { version = "1.35", features = ["full"] }
async-trait = "0.1"
thiserror = "2.0"
serde = { version = "1.0", features = ["derive"] }

# Optional: armature-core for DI integration
armature-core = { path = "../armature-core", version = "0.1.0", optional = true }

[features]
default = []
# Feature for DI integration
di = ["armature-core"]

[dev-dependencies]
tokio-test = "0.4"
```

## Error Handling Pattern

```rust
// src/error.rs
use thiserror::Error;

#[derive(Debug, Error)]
pub enum ModuleError {
    #[error("Configuration error: {0}")]
    Config(String),

    #[error("Connection failed: {0}")]
    Connection(String),

    #[error("Operation failed: {0}")]
    Operation(String),

    #[error(transparent)]
    Io(#[from] std::io::Error),
}

pub type Result<T> = std::result::Result<T, ModuleError>;
```

## Configuration Pattern

```rust
// src/config.rs
use serde::{Deserialize, Serialize};

/// Configuration for the module.
///
/// # Examples
///
/// ```rust
/// use armature_<name>::Config;
///
/// let config = Config::builder()
///     .option1("value")
///     .option2(42)
///     .build();
/// ```
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Config {
    pub option1: String,
    pub option2: i32,
    #[serde(default)]
    pub optional_field: Option<String>,
}

impl Config {
    pub fn builder() -> ConfigBuilder {
        ConfigBuilder::default()
    }

    pub fn from_env() -> ConfigBuilder {
        ConfigBuilder::from_env()
    }
}

#[derive(Debug, Default)]
pub struct ConfigBuilder {
    option1: Option<String>,
    option2: Option<i32>,
    optional_field: Option<String>,
}

impl ConfigBuilder {
    pub fn from_env() -> Self {
        Self {
            option1: std::env::var("MODULE_OPTION1").ok(),
            option2: std::env::var("MODULE_OPTION2").ok().and_then(|v| v.parse().ok()),
            optional_field: std::env::var("MODULE_OPTIONAL").ok(),
        }
    }

    pub fn option1(mut self, value: impl Into<String>) -> Self {
        self.option1 = Some(value.into());
        self
    }

    pub fn option2(mut self, value: i32) -> Self {
        self.option2 = Some(value);
        self
    }

    pub fn optional_field(mut self, value: impl Into<String>) -> Self {
        self.optional_field = Some(value.into());
        self
    }

    pub fn build(self) -> Config {
        Config {
            option1: self.option1.unwrap_or_default(),
            option2: self.option2.unwrap_or(0),
            optional_field: self.optional_field,
        }
    }
}
```

## Service Pattern with DI Integration

```rust
// src/service.rs
use crate::{Config, Result};

/// Main service for the module.
///
/// Supports automatic dependency injection when the `di` feature is enabled.
#[derive(Clone)]
pub struct ModuleService {
    config: Config,
    // Internal state
}

impl ModuleService {
    /// Create a new service with the given configuration.
    pub fn new(config: Config) -> Self {
        Self { config }
    }

    /// Create with default configuration.
    pub fn default() -> Self {
        Self::new(Config::builder().build())
    }

    /// Primary operation of this module.
    pub async fn do_something(&self, input: &str) -> Result<String> {
        // Implementation
        Ok(format!("Processed: {}", input))
    }
}

// DI integration (when feature enabled)
#[cfg(feature = "di")]
mod di {
    use super::*;
    use armature_core::prelude::*;

    impl Provider for ModuleService {
        fn create(_container: &Container) -> std::result::Result<Self, armature_core::Error> {
            Ok(Self::default())
        }
    }
}
```

## Trait Definition Pattern

```rust
// src/traits.rs
use async_trait::async_trait;
use crate::Result;

/// Core trait for module implementations.
///
/// Implement this trait to create custom backends.
#[async_trait]
pub trait Backend: Send + Sync {
    /// Initialize the backend.
    async fn init(&mut self) -> Result<()>;

    /// Perform the main operation.
    async fn execute(&self, input: &str) -> Result<String>;

    /// Clean up resources.
    async fn shutdown(&mut self) -> Result<()>;
}
```

## lib.rs Structure

```rust
//! Armature <Name> Module
//!
//! This module provides <brief description>.
//!
//! # Features
//!
//! - Feature 1

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quinnjr/armature](https://github.com/quinnjr/armature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
