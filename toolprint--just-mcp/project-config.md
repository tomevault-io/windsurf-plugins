---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

just-mcp is a Model Context Protocol (MCP) server that transforms justfiles into AI-accessible automation tools. It monitors directories for justfiles, parses them, and exposes their tasks as MCP tools that AI assistants can discover and execute.

## Essential Development Commands

The justfile has been refactored into a modular system with specialized modules. Use these unified commands:

```bash
# Quick Start & Setup
just quickstart         # Complete setup for new developers
just dev-setup          # Comprehensive development environment setup
just help               # Comprehensive help system with discovery features

# Core Development Workflow
just build [mode]       # Build project (debug/release)
just test [coverage]    # Run tests (with optional coverage)
just format [target]    # Format code (rust/json/markdown/all)
just lint [target] [fix] # Lint code with optional auto-fix
just check [target]     # Combined format + lint + test (quick/full/all)
just pre-commit         # Full validation before committing

# Workflow Automation
just workflow quick     # Quick development workflow (format + lint + test)
just workflow full      # Full workflow with coverage and release build  
just workflow commit    # Commit-ready workflow (runs pre-commit)
just ci                 # Complete CI/CD checks locally

# Installation & Release
just install [features] # Install binaries (with optional vector-search)
just release-info       # Show release binary information
just clean              # Clean build artifacts

# Discovery & Navigation
just list               # List all recipes organized by groups
just groups             # List all recipe groups
just summary            # Compact recipe list
just help-topics        # Show all available help topics
```

## Modular Justfile Architecture

The justfile has been transformed from a monolithic 870-line file into a modular system:

### **Main Justfile** (`justfile`)

- **Comprehensive help system** with progressive disclosure
- **Unified delegation commands** providing consistent interfaces
- **Workflow recipes** combining multiple operations
- **Discovery features** for easy navigation

### **Specialized Modules** (`just/` directory)

- **`rust.just`** - Rust development (build, test, lint, docs, install)
- **`setup.just`** - Project setup and tool installation
- **`vector.just`** - Vector search demos and utilities
- **`docker.just`** - Docker/Dagger CI/CD operations
- **`release.just`** - Cross-platform release and deployment
- **`common.just`** - Shared utilities and error handling

### **Module-Specific Commands**

```bash
# Access module-specific functionality directly:
just build-rust-release          # Direct Rust build
just setup-brew                  # Install development tools
just demo-vector-search          # Vector search demonstration
just dagger-ci                   # Run Dagger CI pipeline
just zigbuild-release v1.0.0     # Cross-platform release

# Or use module help:
just rust-help                   # Rust development commands
just docker-help                 # Docker/Dagger commands
just vector-help                 # Vector search commands
```

## Architecture Overview

### Core Flow: Justfile → MCP Tool

1. **Watcher** monitors directories for justfile changes
2. **Parser** extracts tasks with parameters, dependencies, descriptions
3. **Registry** converts tasks to MCP tools with JSON schemas
4. **Server** exposes tools via MCP protocol over stdio
5. **Executor** runs just commands when tools are called

### Key Architectural Decisions

- **Async Everything**: Built on Tokio for concurrent operations
- **Channel-Based Communication**: Components communicate via broadcast channels for decoupling
- **Security by Design**: All inputs validated, paths restricted, resources limited
- **Tool Naming**: Format is `just_<task>@<name>` or `just_<task>_<full_path>`
- **AST-Based Parsing**: Tree-sitter parser provides accurate syntax understanding

### Module Interactions

```text
main.rs → Server → Registry ← Watcher
                ↓            ↓
             Handler      Parser (AST/CLI/Regex)
                ↓
            Executor → Security + ResourceLimits
```

## AST Parser (Tree-sitter Integration)

just-mcp now uses an AST-based parser powered by Tree-sitter as the primary parsing method, with automatic fallback to CLI and regex parsers for compatibility.

### Parser Priority System

The `EnhancedJustfileParser` implements a three-tier fallback system:

1. **AST Parser** (Default) - Most accurate, handles all Just syntax features
2. **CLI Parser** - Uses `just --summary` for recipe discovery
3. **Regex Parser** - Basic pattern matching for simple justfiles

### Building with AST Support

```bash
# Build with AST parser support (recommended)
cargo build --features ast-parser

# Build with all features including AST parser
cargo build --features all

# Test AST parser functionality
cargo test --features ast-parser ast_parser
```

### AST Parser Benefits

- **Complete Syntax Support**: Handles multiline strings, complex expressions, all attributes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [toolprint/just-mcp](https://github.com/toolprint/just-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
