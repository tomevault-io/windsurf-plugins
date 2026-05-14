---
trigger: always_on
description: **Technical Peer Collaboration Mode:**
---

# C++ MCP Server Project

## 🤝 Collaboration Style Preference

**Technical Peer Collaboration Mode:**

- **Direct, honest technical opinions** - Call out anti-patterns and architectural issues directly
- **Equal footing partnership** - Co-architects debating design decisions and building consensus
- **Constructive disagreement encouraged** - Push back on current approaches while offering concrete alternatives
- **Shared problem-solving** - Build on each other's ideas collaboratively rather than just responding to requests
- **Practical trade-off focus** - Stay grounded in real maintainability vs complexity decisions
- **Casual but substantive tone** - Technical depth with informal, enthusiastic communication

**What works:** Instead of helpful assistant mode, engage as **technical teammate** who can disagree, offer strong opinions, and get excited when ideas align. Focus on sustainable architecture decisions together.

## Project Overview

This is a **C++ MCP (Model Context Protocol) server** implemented in Rust that bridges AI agents with C++ LSP tools (primarily clangd). The goal is to provide AI agents with the same semantic code understanding capabilities that human C++ developers rely on through intellisense.

## Project Context & Rationale

- **Problem**: AI agents use different approaches to browse code - some rely on text search, others on LSP integration
- **Target**: Large C++ codebases with heavy preprocessor usage where humans rely on intellisense
- **Solution**: Bridge AI agents with C++ LSP tools to provide semantic understanding beyond text search
- **Technology Choice**: Rust for resource efficiency when handling large compilation databases

## Current Implementation Status

### ✅ Completed

- Full MCP server implementation with rust-mcp-sdk
- CMake project analysis and build directory management (`list_build_dirs`)
- Comprehensive C++ symbol search with project boundary detection (`search_symbols`)
- Deep symbol analysis with inheritance and call hierarchy (`analyze_symbol_context`)
- Clangd LSP client with lifecycle management and indexing progress tracking
- Project vs external symbol filtering using compilation database analysis
- Structured JSON responses with comprehensive error handling
- CI/CD pipeline with build, tests, clippy, and security audit
- **Python CLI Tool (`mcp-cli.py`)** - Standalone command-line interface for easy MCP server interaction
- **FIXED: Result limiting architecture** - Proper client-side limiting preserving clangd ranking

### 🔄 Current Architecture

```
src/
├── main.rs              // MCP server entry point with stdio transport
├── logging.rs           // Structured logging and MCP message tracing
├── test_utils.rs        // Testing utilities and helpers
├── clangd/              // Clangd LSP integration layer
│   ├── mod.rs           // Module exports
│   ├── session.rs       // Clangd session lifecycle management
│   ├── session_builder.rs // Session configuration and builder pattern
│   ├── file_manager.rs  // LSP document and file operations
│   ├── config.rs        // Clangd configuration management
│   ├── version.rs       // Clangd version detection and compatibility
│   ├── testing.rs       // Testing utilities for clangd integration
│   ├── error.rs         // Clangd-specific error handling
│   └── index/           // Indexing and progress monitoring
│       ├── mod.rs       // Index module exports
│       ├── monitor.rs   // Real-time indexing progress tracking
│       ├── hash.rs      // Index hash computation and validation
│       └── project_index.rs // Project-specific index management
├── lsp/                 // LSP protocol implementation
│   ├── mod.rs           // Module exports
│   ├── client.rs        // JSON-RPC LSP client communication
│   ├── protocol.rs      // LSP types and protocol definitions
│   ├── framing.rs       // JSON-RPC message framing
│   ├── jsonrpc_utils.rs // JSON-RPC utilities and helpers
│   ├── traits.rs        // LSP client traits and interfaces
│   └── testing.rs       // LSP testing utilities
├── io/                  // I/O and transport layer
│   ├── mod.rs           // Module exports
│   ├── process.rs       // Process management and lifecycle
│   └── transport.rs     // Transport abstractions
├── project/             // Multi-provider project analysis
│   ├── mod.rs           // Module exports
│   ├── workspace.rs     // Project workspace management
│   ├── workspace_session.rs // Workspace session state management
│   ├── scanner.rs       // Multi-provider project scanning
│   ├── provider.rs      // Build system provider trait
│   ├── component.rs     // Project component abstraction
│   ├── cmake_provider.rs // CMake project provider
│   ├── meson_provider.rs // Meson project provider
│   ├── compilation_database.rs // Compilation database handling
│   └── error.rs         // Project-specific error handling
└── mcp_server/          // MCP server implementation
    ├── mod.rs           // Module exports
    ├── server.rs        // Core MCP server handler
    ├── server_helpers.rs // Server utility functions
    └── tools/           // MCP tool implementations
        ├── analyze_symbols.rs  // Deep symbol analysis
        ├── search_symbols.rs   // C++ symbol search with filtering

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mpsm/mcp-cpp](https://github.com/mpsm/mcp-cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
