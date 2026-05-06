---
trigger: always_on
description: Vectorizer is a high-performance, in-memory vector database written in Rust, designed for AI-driven applications requiring fast similarity search and vector operations. This document provides specific instructions for GitHub Copilot to generate consistent, high-quality code that follows the project's standards and patterns.
---

# GitHub Copilot Instructions for Vectorizer

## Overview

Vectorizer is a high-performance, in-memory vector database written in Rust, designed for AI-driven applications requiring fast similarity search and vector operations. This document provides specific instructions for GitHub Copilot to generate consistent, high-quality code that follows the project's standards and patterns.

## ⚠️ CRITICAL ARCHITECTURE NOTE

**REST-First Architecture with MCP Support**

- **REST API**: Primary HTTP interface managing all collections and vector data
- **MCP Server**: Model Context Protocol interface for AI assistants
- **Core Engine**: Rust-based vector store with HNSW indexing

**RULE: REST and MCP must have EXACTLY the same functionality**

When adding features:
1. **Implement in core engine first** (business logic)
2. **Add REST endpoints** (HTTP interface)
3. **Add MCP tools** (AI assistant interface)

**NEVER implement features only in MCP!** Both layers must be kept in sync.

**Architecture**: REST + MCP unified server system

## 🚨 RUST EDITION REQUIREMENT - CRITICAL

**This project MANDATORILY uses Rust Edition 2024**

### Non-Negotiable Requirements:
- **Cargo.toml edition field**: MUST be `"2024"` (never change to 2021 or other editions)
- **Build environment**: Must use Rust toolchain with Edition 2024 support
- **Dependencies**: All dependencies must be compatible with Edition 2024

### Why Edition 2024:
- Required for advanced async patterns used throughout the codebase
- Enables memory optimizations and performance features
- Provides access to latest language improvements

### 🚫 NEVER CHANGE THIS SETTING:
```toml
[package]
edition = "2024"  # ← This MUST stay as "2024"
```

**Consequences of changing edition:**
- Compilation failures for Edition 2024 specific code
- Loss of critical language features
- Performance regressions
- Dependency incompatibility

## Core Principles

### Code Quality Standards
- **Readability First**: Generate self-documenting code with clear, descriptive names
- **Consistency**: Follow established patterns throughout the codebase
- **Performance**: Consider performance implications, especially in hot paths
- **Maintainability**: Write code that can be easily understood and modified

### Documentation Requirements
- **Module docs**: Start with `//!` for crate/module-level documentation
- **Item docs**: Use `///` for all public functions, structs, enums, traits, and their fields
- **Parameter docs**: Document all parameters with their types and purposes
- **Return docs**: Explain what functions return and under what conditions
- **Error docs**: Document all error conditions and edge cases
- **Examples**: Include code examples where helpful

## Language-Specific Patterns

### Rust Idioms and Patterns

#### Struct and Enum Definitions
```rust
// Always include these derives in this order for data structures
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct Vector {
    /// Unique identifier for the vector
    pub id: String,
    /// The vector data
    pub data: Vec<f32>,
    /// Optional payload associated with the vector
    pub payload: Option<Payload>,
}

// API enums should use lowercase serialization
#[derive(Debug, Clone, Copy, Serialize, Deserialize)]
#[serde(rename_all = "lowercase")]
pub enum DistanceMetric {
    Cosine,
    Euclidean,
    DotProduct,
}

// Implement Display for user-facing enums
impl fmt::Display for DistanceMetric {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        match self {
            DistanceMetric::Cosine => write!(f, "cosine"),
            DistanceMetric::Euclidean => write!(f, "euclidean"),
            DistanceMetric::DotProduct => write!(f, "dot_product"),
        }
    }
}
```

#### Error Handling
```rust
// Use custom error types with meaningful messages
#[derive(thiserror::Error, Debug)]
pub enum VectorizerError {
    #[error("Collection not found: {0}")]
    CollectionNotFound(String),

    #[error("Dimension mismatch: expected {expected}, got {actual}")]
    DimensionMismatch { expected: usize, actual: usize },

    #[error("IO error: {0}")]
    IoError(#[from] std::io::Error),

    #[error("JSON error: {0}")]
    JsonError(#[from] serde_json::Error),
}

pub type Result<T> = std::result::Result<T, VectorizerError>;

// Error propagation patterns
pub fn create_collection(name: &str, config: CollectionConfig) -> Result<(), VectorizerError> {
    // Validate input first
    if name.is_empty() {
        return Err(VectorizerError::InvalidConfiguration {
            message: "Collection name cannot be empty".to_string(),
        });
    }

    // Use ? operator for error propagation
    let collection = Collection::new(name.to_string(), config)?;
    self.collections.insert(name.to_string(), Arc::new(collection));

    Ok(())
}
```

#### Concurrency Patterns
```rust
// Shared ownership across threads

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hivellm/vectorizer](https://github.com/hivellm/vectorizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
