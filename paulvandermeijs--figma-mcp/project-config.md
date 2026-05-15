---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building and Running
```bash
# Development build and check
cargo check
cargo build

# Release build (for distribution)
cargo build --release

# Run the MCP server (requires FIGMA_TOKEN environment variable)
FIGMA_TOKEN="your_token" cargo run

# Run with debug logging
RUST_LOG=debug FIGMA_TOKEN="your_token" cargo run
```

### Testing
```bash
# Run all tests
cargo test

# Run specific test module
cargo test url_parsing
cargo test api_client

# Run tests with output and logging
cargo test -- --nocapture
RUST_LOG=debug cargo test
```

### Code Quality
```bash
# Format code
cargo fmt

# Run linter
cargo clippy
```

## High-Level Architecture

This is a **Model Context Protocol (MCP) server** that provides AI assistants with tools to access Figma files via Figma's API. The server focuses on file-based operations using a two-step workflow: first parse URLs to extract file keys, then use those keys with file operation tools. Exported images are automatically available as MCP resources with base64-encoded content.

### Core Architecture Pattern

The codebase follows a **layered architecture** with clear separation between:

1. **Protocol Layer** (`src/server.rs`): MCP server implementation using the `rmcp` crate
2. **Business Logic Layer** (`src/figma/`): Figma-specific functionality  
3. **Infrastructure Layer**: HTTP client, error handling, URL parsing

### Key Components

**URL Parser (`src/figma/url_parser.rs`)**
- Central component that extracts file IDs from various Figma URL formats
- Uses regex patterns to handle different URL variations (file and design URLs)
- Returns structured `FigmaUrlInfo` with parsed components
- Critical for the URL-first approach due to Figma API limitations

**API Client (`src/figma/client.rs`)**
- Returns `serde_json::Value` instead of typed structs for flexibility
- Handles Figma authentication via personal access tokens
- Implements file-focused Figma API endpoints (files, nodes, image export, user info)
- Comprehensive error handling for API failures and rate limiting (60 req/min)

**Image Cache (`src/figma/image_cache.rs`)**
- Manages exported images as MCP resources
- Thread-safe storage using `Arc<RwLock<HashMap>>`
- Tracks Figma URLs, export metadata, and cached image data
- Handles URL expiration and image data caching

**MCP Server (`src/server.rs`)**
- Implements 6 MCP tools using `#[tool]` attribute macros focused on file operations
- Uses typed parameter structs with `#[derive(JsonSchema)]` for proper MCP Inspector integration
- All tools follow pattern: `Parameters<StructName>` for parameter binding
- Returns JSON strings via `CallToolResult::success()`
- Implements resource handlers for listing and reading exported images
- Downloads and base64-encodes images on demand

**Error Handling (`src/error.rs`)**
- Custom error enum covering all failure modes (API, network, auth, JSON, URL parsing)
- User-friendly error messages throughout the application
- Proper error propagation using `?` operator and `thiserror`

### MCP Tools Architecture

All tools focus on file-based operations with a clear separation between URL parsing and file operations:

**URL Parsing Tool**:
- `parse_figma_url` - Parse URLs to extract file keys and node information

**File Operation Tools** (require file key from `parse_figma_url`):
- `get_file` - Complete file data extraction using file key with depth control (default: 1)
- `get_file_nodes` - Specific node data using file key with depth control (default: 1)
- `export_images` - Image export using file key

**Utility Tools**:
- `get_me` - Authentication testing
- `help` - Usage instructions

### Parameter Schema System

The server uses a sophisticated parameter schema system:
- Each tool has a dedicated parameter struct (e.g., `ParseUrlRequest`, `GetFileRequest`)
- Structs derive `JsonSchema` for MCP Inspector integration
- Field-level documentation via `#[schemars(description = "...")]`
- Optional parameters use `Option<T>`
- Parameters bound via `Parameters<StructName>` pattern in tool signatures

### Data Flow Pattern

1. **Input**: User provides Figma URL through MCP client
2. **Parse**: `parse_figma_url` tool extracts file key and node information
3. **File Operations**: User provides file key to file operation tools (`get_file`, `get_file_nodes`, `export_images`) with optional depth control
4. **Fetch**: API client makes authenticated requests to Figma using file key and depth parameter
5. **Return**: Raw JSON data returned as flexible `serde_json::Value` (limited by depth to manage response size)
6. **Navigation**: For large files, use recursive calls with specific node IDs to explore deeper
7. **Output**: JSON serialized and returned via MCP protocol

This design enforces a **clear separation of concerns** and **response size management** - URL parsing is separate from file operations, and depth control prevents token limit issues while enabling incremental exploration of large Figma files.

### Depth Parameter and Response Size Management

The server implements Figma's depth parameter to control response size and prevent LLM token limit issues:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paulvandermeijs/figma-mcp](https://github.com/paulvandermeijs/figma-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
