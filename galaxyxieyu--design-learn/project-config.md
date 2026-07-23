---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Design-Learn is a web design extraction and analysis system with three main components:
1. **Chrome Extension** - Zero-dependency browser plugin for extracting page snapshots (HTML/CSS/images/fonts)
2. **VSCode Extension** - Management UI and local server launcher
3. **Design-Learn Server** - Node.js backend providing REST API, MCP tools, and data storage

The system enables users to capture web design snapshots, analyze them with AI, and manage design resources through multiple interfaces.

## Architecture

### Multi-Client Single-Server Model

```
Chrome Extension ──┐
                   ├──> HTTP API (port 3100)
VSCode Extension ──┤    - /api/health
                   │    - /api/import/*
Claude Code ───────┘    - /mcp (SSE)
                        - /api/designs
                        - /api/tasks

                   Design-Learn Server
                   (SQLite + File Storage)
```

### Data Flow

1. **Browser Extraction**: Chrome extension captures page snapshot → sends to `/api/import/browser`
2. **Server Processing**: Extraction pipeline processes snapshot → stores in SQLite + file system
3. **MCP Access**: Claude Code queries designs via MCP tools (`list_designs`, `get_design`, etc.)
4. **VSCode Management**: VSCode extension manages server lifecycle and displays snapshots

### Storage Architecture

- **SQLite Database** (`data/database.sqlite`): Metadata for designs, versions, components, rules, tasks
- **File System** (`data/designs/`): JSON metadata, styleguides, snapshots, component code
- **Hybrid Approach**: SQLite for queries, files for large content

## Development Commands

### Server Development

```bash
# Install dependencies (must be in server directory)
cd server
npm install

# Rebuild native modules if needed
npm rebuild better-sqlite3

# Start server (default port 3100)
node server/src/server.js

# Start with custom port
PORT=3200 node server/src/server.js

# Start via CLI
node server/src/cli.js

# MCP stdio mode (for Claude Code integration)
node server/src/stdio.js
```

### VSCode Extension Development

```bash
cd vscode-extension

# Install dependencies
npm install

# Compile TypeScript
npm run compile

# Watch mode for development
npm run watch

# Package extension (warnings about LICENSE and file count are expected and can be ignored)
npx vsce package --out ../dist/design-learn-1.0.2.vsix

# Install extension
code --install-extension ../dist/design-learn-1.0.2.vsix --force

# CRITICAL: VSCode caches webview content. After reinstalling:
# 1. Completely quit VSCode (Cmd+Q on macOS, not just close window)
# 2. Reopen VSCode
# 3. Failure to do this will result in old cached code running

# Development mode (recommended for testing - no need to package)
# 1. Open vscode-extension folder in VSCode
# 2. Press F5 to launch Extension Development Host
# 3. Test in the new window, set breakpoints in original window
```

### Chrome Extension Development

No build step required - load directly in Chrome:
1. Navigate to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked"
4. Select the `chrome-extension/` directory

### Testing & Verification

```bash
# Backend verification (from repository root)
./scripts/verify-backend.sh

# Custom port verification
PORT=3100 ./scripts/verify-backend.sh

# Manual API testing
curl http://localhost:3100/api/health
curl http://localhost:3100/api/designs
```

## Key Technical Patterns

### Server Entry Points

- **HTTP Server** (`src/server.js`): Main entry point, handles all HTTP/WebSocket/MCP traffic
- **MCP Stdio** (`src/stdio.js`): Stdio transport for Claude Code MCP integration
- **CLI** (`src/cli.js`): Command-line interface for server management

### MCP Tool Implementation

MCP tools are defined in [server/src/mcp/index.js](server/src/mcp/index.js):

```javascript
// Tool registration pattern
server.registerTool(toolName, schema, handler);

// Available tools:
// - ping: Health check
// - list_designs: List all designs with optional limit
// - search_designs: Search by keyword/tags/URL
// - get_design: Fetch design metadata by ID
// - get_rules: Fetch version rules (colors/typography/spacing)
// - list_versions, get_version: Version management
// - list_components, get_component, get_component_preview: Component access
```

### Storage Layer Pattern

Storage operations follow a consistent pattern in [server/src/storage/index.js](server/src/storage/index.js):

```javascript
// 1. Normalize input data
const design = normalizeDesign(input);

// 2. Write to file system
await writeJson(designPath, design);

// 3. Insert/update SQLite metadata
db.prepare('INSERT INTO designs ...').run(...);

// 4. Update indexes
await writeDesignIndex(db, dataDir);
```

### Extraction Pipeline

The extraction pipeline ([server/src/pipeline/index.js](server/src/pipeline/index.js)) handles:
- Job queue management with status tracking
- SSE progress streaming to clients
- Browser import (from Chrome extension)
- URL import (with optional Playwright for server-side extraction)

### Task Management System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GalaxyXieyu/Design-Learn](https://github.com/GalaxyXieyu/Design-Learn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
