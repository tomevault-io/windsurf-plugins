---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Yii2 AI Boost** is a Model Context Protocol (MCP) server that integrates with Yii2 applications to provide AI assistants with tools for framework introspection, database inspection, and application guidelines. It implements MCP v2025-11-25 with JSON-RPC 2.0 over STDIO transport.

The package is installable as a Composer dependency and provides:
- **16 Tools** for introspection (application info, database schema, database query, config, routes, components, logs, semantic search, model inspector, validation rules, console command inspector, migration inspector, widget inspector, performance profiler, tinker, env inspector)
- **Installation Wizard** for IDE integration (Claude Code, VS Code, Cursor, PhpStorm)
- **Comprehensive Logging** across multiple levels (startup, requests, errors, transport)

## Development Commands

```bash
# Run all tests
composer test

# Generate code coverage report
composer test:coverage

# Check PSR-12 code style
composer cs-check

# Auto-fix code style
composer cs-fix

# Run PHPStan static analysis (level 8)
composer analyze

# Start MCP server (for manual testing)
php yii boost/mcp

# View installation status
php yii boost/info

# Run installation wizard
php yii boost/install

# Update guidelines
php yii boost/update
```

## High-Level Architecture

The codebase follows a **layered, modular design** with clear separation of concerns:

```
CLI Commands Layer
    ↓ (Yii2 Bootstrap)
MCP Server Layer (JSON-RPC dispatcher)
    ├─ Tools (Domain logic - introspection)
    └─ Transports (I/O protocol)
        ↓
Yii2 Application Integration
    └─ Database, Config, Routes, Components
```

### Core Layers

1. **Bootstrap/Commands Layer** (`src/Commands/`, `src/Bootstrap.php`)
   - Entry points via Yii2 console commands
   - `BoostController` - Main dispatcher
   - `McpController` - Starts MCP server with proper logging setup
   - `InstallController` - Installation wizard
   - `InfoController` - Status display
   - `UpdateController` - Guidelines management

2. **MCP Server Layer** (`src/Mcp/Server.php`)
   - JSON-RPC 2.0 protocol handler
   - Dispatches requests to tools/resources
   - Manages tool and resource registration
   - Handles error responses and logging

3. **Tools Layer** (`src/Mcp/Tools/`)
   - Independent, pluggable introspection tools
   - All extend `BaseTool` for consistency
   - Automatic sanitization of sensitive data
   - Support JSON Schema input validation
   - Current tools: ApplicationInfo, DatabaseSchema, DatabaseQuery, ConfigAccess, RouteInspector, ComponentInspector, LogInspector, SemanticSearch, ModelInspector, ValidationRules, ConsoleCommandInspector, MigrationInspector, WidgetInspector, PerformanceProfiler, Tinker, EnvInspector

5. **Search Layer** (`src/Mcp/Search/`)
   - FTS5-powered search using separate SQLite database
   - `MarkdownSectionParser` - Splits markdown into H2 sections
   - `SearchIndexManager` - FTS5 index management (raw PDO, not Yii2 DB)
   - `GitHubGuideDownloader` - Fetches Yii2 guide from GitHub, caches locally
   - Index location: `@runtime/boost/search.db`

4. **Transport Layer** (`src/Mcp/Transports/StdioTransport.php`)
   - STDIO communication (reads STDIN, writes STDOUT)
   - Completely decoupled from business logic
   - Currently STDIO-only (no transport abstraction layer)

### Critical Architectural Patterns

**Plugin Registration Pattern**: Tools are explicitly registered in `Server::registerTools()` - easy to add new tools without modifying core logic.

**Dispatch/Router Pattern**: `Server::dispatch()` routes JSON-RPC method calls to specific handlers. Each method has a dedicated handler function.

**Template Method Pattern**: `BaseTool` provides common functionality (data sanitization, database discovery, logging) that all tools inherit.

**Callback/Handler Pattern**: Transport uses callbacks to decouple I/O from business logic.

## Request Flow (STDIN → STDOUT)

```
1. Client sends JSON-RPC request to STDIN
2. StdioTransport::listen() reads line via fgets()
3. Server::handleRequest() parses and validates JSON
4. Request logged to mcp-requests.log
5. Server::dispatch() routes method to handler
6. Handler executes tool or reads resource
7. BaseTool::sanitize() removes sensitive data
8. Response formatted as JSON-RPC 2.0
9. Response logged to mcp-requests.log
10. Response written to STDOUT
```

**Key Detail**: STDOUT reserved for JSON-RPC responses only. All logging/errors go to STDERR and log files to prevent mixing protocol messages with debug output.

## Adding New Tools

1. Create class in `src/Mcp/Tools/MyTool.php` extending `BaseTool`
2. Implement: `getName()`, `getDescription()`, `getInputSchema()`, `execute()`
3. Add class to `$toolClasses` array in `Server::registerTools()`
4. Tool auto-available via MCP protocol - no other changes needed
5. Use `sanitize()` method for sensitive data
6. Update README and `CLAUDE.md`

## Configuration Files

**`.mcp.json`** - IDE integration (auto-generated by installer)
- Specifies PHP command and arguments for IDE
- Created for Claude Code, VS Code, Cursor, PhpStorm


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codeChap/mcp-server-yii2-ai-boost](https://github.com/codeChap/mcp-server-yii2-ai-boost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
