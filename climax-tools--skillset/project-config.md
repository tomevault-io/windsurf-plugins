---
trigger: always_on
description: **Skillset** is a Rust-based CLI package manager designed specifically for coding agent skills. It provides npm-like semantics for managing skills across multiple coding agent frameworks while abstracting away the complexity of different agent conventions.
---

# AGENTS.md - Skillset CLI Architecture and Integration Guide

## Tool Overview

**Skillset** is a Rust-based CLI package manager designed specifically for coding agent skills. It provides npm-like semantics for managing skills across multiple coding agent frameworks while abstracting away the complexity of different agent conventions.

### Scope and Purpose

- **Primary Goal**: Simplify skill discovery, installation, and management for AI agent developers
- **Target Audience**: Node.js/TypeScript developers, Python developers working with AI agents
- **Use Cases**: Development workflows, CI/CD pipelines, skill sharing, and agent ecosystem integration

### Key Problems Solved

1. **Fragmented Agent Ecosystem**: Different frameworks (Auto-GPT, LangChain, custom) have incompatible skill formats
2. **Complex Installation**: No unified way to install skills across frameworks
3. **Configuration Management**: Projects need to track skills, dependencies, and framework compatibility
4. **Distribution**: No standard way to package and share agent skills

### Design Constraints

- **Framework Agnostic**: Must work with existing and future agent frameworks
- **Developer Friendly**: Familiar CLI patterns and configuration formats
- **Extensibility**: Easy to add new frameworks, sources, and features
- **Production Ready**: Robust error handling, configuration management, and caching

## Table of Contents

1. [Core Architecture Overview](#core-architecture-overview)
2. [Pluggable Sources System](#pluggable-sources-system)
3. [Convention System Architecture](#convention-system-architecture)
4. [Project Structure Patterns](#project-structure-patterns)
5. [CLI Design and Semantics](#cli-design-and-semantics)
6. [Integration Guidelines for Agent Frameworks](#integration-guidelines-for-agent-frameworks)
7. [Development Guidelines](#development-guidelines)
8. [Code Examples and Patterns](#code-examples-and-patterns)
9. [Architecture Decision Rationale](#architecture-decision-rationale)

---

## Core Architecture Overview

Skillset follows a modular architecture with clear separation of concerns:

### Key Design Principles

1. **Orthogonal Configuration**: Agent conventions are configured separately from skill definitions
2. **Pluggable Extensibility**: Both sources and conventions can be easily extended
3. **CLI-First Design**: Command-line interface drives all operations
4. **Async-First**: All I/O operations are asynchronous

### Module Organization

```
├── src/
│   ├── cache/                   # User-wide cache infrastructure
│   ├── cli/                    # CLI interface and command handling
│   ├── sources/                 # Pluggable source implementations
│   ├── conventions/              # Agent framework conventions
│   ├── config/                  # Configuration management
│   ├── skill/                   # Core skill data structures
│   ├── registry/                 # OCI registry operations
│   └── error.rs                # Centralized error handling
└── AGENTS.md                   # This architectural documentation
```

---

## Pluggable Sources System

### SkillSource Trait (`src/sources/mod.rs`)

All skill sources implement the `SkillSource` trait:

```rust
#[async_trait]
pub trait SkillSource: Send + Sync {
    async fn fetch(&self, reference: &str) -> Result<FetchedSkill>;
    async fn get_metadata(&self, reference: &str) -> Result<SkillMetadata>;
    fn source_type(&self) -> SourceType;
}
```

#### Supported Source Types

1. **Git Sources** (`src/sources/git.rs`)
   - **User-Wide Cache**: Clone to `~/.cache/skillset/git/checkouts/{skill-name}/`
   - **Metadata Tracking**: JSON files in `~/.cache/skillset/metadata/`
   - Parse git URLs and references
   - Handle branches and tags
   - Extract skill content from repository root
   - **Cache-Aware**: Initialize with shared `CachePaths` infrastructure

2. **OCI Sources** (`src/sources/oci.rs`) - *Not Yet Implemented*
   - Pull from OCI registries (Docker Hub, GitHub Container Registry, etc.)
   - Support ORAS-like artifact format
   - Handle authentication and layer manifests
   - Extract skill content from OCI layers
   - **Future Extensibility**: Cache infrastructure ready for OCI sources

3. **Local Sources** (`src/sources/local.rs`) - *Not Yet Implemented*
   - Load skills from local filesystem paths
   - Useful for development and testing
   - Symlink or copy content to organized directories
   - **Future Extensibility**: Can implement caching with shared CachePaths

#### Source Registry Pattern

```rust
pub struct SourceRegistry {
    sources: HashMap<String, Box<dyn SkillSource>>,
}

impl SourceRegistry {
    pub fn new() -> Result<Self> {
        let mut sources = HashMap::new();
        
        // Sources are cache-aware from initialization
        sources.insert("git".to_string(), Box::new(GitSource::new()?) as Box<dyn SkillSource>);
        
        Ok(Self { sources })
    }
    
    pub fn register(&mut self, source: Box<dyn SkillSource>) {
        let source_type = source.source_type();
        let type_name = match source_type {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [climax-tools/skillset](https://github.com/climax-tools/skillset) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
