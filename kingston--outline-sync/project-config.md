---
trigger: always_on
description: This document outlines the coding standards and conventions used in the outline-sync project. These guidelines ensure consistency and help maintain high code quality.
---

# Coding Guidelines for outline-sync

This document outlines the coding standards and conventions used in the outline-sync project. These guidelines ensure consistency and help maintain high code quality.

## Project Overview

**outline-sync** is a bidirectional synchronization tool that enables 2-way sync between Outline (a documentation/wiki platform) and your local filesystem. It also includes MCP (Model Context Protocol) support for AI assistant integration.

### Key Features

- Downloads documentation from Outline to local markdown files
- Uploads local markdown changes back to Outline
- Handles embedded images (download/upload)
- Preserves document hierarchy and metadata
- Supports selective sync with collection filtering
- MCP server for AI assistant integration

## Project Structure

- **Single Package**: Not a monorepo, but follows similar conventions
- **Package Manager**: pnpm 10+
- **Node Version**: 20+ (specified in engines, Volta pinned to 22.16.0)
- **Module System**: ESM only (`"type": "module"` in package.json)

## Code Structure

### Directory Organization

```
src/
├── commands/           # CLI command implementations
│   ├── annotate.ts     # Annotates markdown files with AI-generated descriptions
│   ├── download.ts     # Downloads collections from Outline to local files
│   ├── mcp.ts          # Starts MCP server for AI integration
│   └── upload.ts       # Uploads local changes back to Outline
│
├── services/           # Core business logic
│   ├── attachments.ts  # Image/attachment handling and markdown processing
│   ├── documents.ts    # Document fetching with hierarchy building
│   ├── langchain.ts    # LangChain integration for AI features
│   ├── outline.ts      # Main Outline API client using OpenAPI types
│   ├── output-files.ts # File system operations for collections
│   └── generated/      # OpenAPI TypeScript definitions
│       └── outline-openapi.d.ts
│
├── mcp/               # Model Context Protocol server
│   ├── server.ts      # MCP server implementation (stdio/SSE transports)
│   ├── resources/     # MCP resource handlers
│   │   ├── documents.ts # Document resource access
│   │   └── index.ts   # Resource exports
│   ├── tools/         # MCP tool implementations
│   │   ├── index.ts   # Tool exports
│   │   └── list-collections.ts # List available collections
│   ├── types.ts       # MCP-specific type definitions
│   └── utils/         # MCP test utilities
│       └── mcp-runner.test-helper.ts # MCP test runner
│
├── types/             # TypeScript type definitions
│   ├── collections.ts # Collection types
│   ├── config.ts      # Configuration schemas using Zod validation
│   ├── documents.ts   # Document-related types
│   └── index.ts       # Type re-exports
│
├── utils/             # Utility functions
│   ├── collection-filter.ts # Collection filtering logic
│   ├── config.ts      # Configuration loading and validation
│   ├── file-manager.ts # File system operations (frontmatter handling)
│   ├── file-names.ts  # Safe filename generation
│   ├── file-transfer.ts # HTTP file upload/download utilities
│   ├── find-nearest-package-json.ts # Package.json location utility
│   ├── handle-not-found-error.ts # Error handling utilities
│   └── version.ts     # Version management
│
├── tests/             # Test utilities and helpers
│   ├── config.test-helper.ts # Configuration mock helpers
│   └── factories.test-helper.ts # Test data factories
│
├── constants/         # Application constants
│   └── concurrency.ts # Concurrency limits for API requests
│
├── __mocks__/         # Manual mocks for testing
│   ├── fs.cts         # File system mocks
│   └── fs/
│       └── promises.cts # Async file system mocks
│
└── index.ts           # Main entry point with CLI setup
```

### Key Workflows

1. **Download Flow**:
   - Fetches collections from Outline API
   - Builds document hierarchy recursively
   - Downloads document content and embedded images
   - Writes markdown files with frontmatter metadata

2. **Upload Flow**:
   - Scans local markdown files
   - Parses frontmatter to identify documents
   - Uploads new images to Outline
   - Creates or updates documents via API

3. **MCP Integration**:
   - Provides AI assistants with direct access to synced documentation
   - Supports stdio and SSE transports
   - Exposes document resources through standardized protocol

### Configuration

The project uses a JSON-based configuration file with Zod validation:

- `outline-sync.config.json` - Main configuration file
- Supports collection filtering, output directories, and sync behavior
- Environment variables can override config values

## File Naming Conventions

- **Files**: Use kebab-case for all file names (e.g., `system-info.ts`, `eslint.config.js`)
- **Directories**: Use kebab-case for directory names
- **Test Files**: Use descriptive suffixes:
  - `.unit.test.ts` for unit tests
  - `.int.test.ts` for integration tests
  - Tests should be collocated with source files in the same directory

## Import/Export Rules

### ESM-Style Imports

- **Always use .js extensions** in import statements, even when importing TypeScript files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kingston/outline-sync](https://github.com/kingston/outline-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
