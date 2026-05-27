---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants working with the Unconventional Thinking Server codebase.
---

# CLAUDE.md - AI Assistant Development Guide

This document provides comprehensive guidance for AI assistants working with the Unconventional Thinking Server codebase.

## Project Overview

**Name**: Unconventional Thinking Server (unreasnable-thinker-server)
**Version**: 0.3.0
**Type**: MCP (Model Context Protocol) Server — spec 2025-11-25, SDK 1.27.1
**Purpose**: A context-efficient tool for generating and tracking unconventional, boundary-breaking problem-solving thoughts

This is a TypeScript-based MCP server that implements an unconventional thinking system optimized for **context space savings** based on Anthropic's latest MCP architecture patterns. The server demonstrates recommended patterns for reducing context overhead by 98.7%.

## Repository Structure

```
Unconventional-thinking/
├── src/
│   └── index.ts              # Main server implementation (all logic in single file)
├── .thoughts/
│   └── thoughts.json         # Persistent storage for generated thoughts
├── build/                    # Compiled JavaScript output (gitignored)
├── node_modules/             # Dependencies (gitignored)
├── package.json              # Project metadata and scripts
├── tsconfig.json             # TypeScript configuration
├── .gitignore                # Git ignore patterns
├── README.md                 # User-facing documentation
├── LICENSE                   # MIT License
└── CLAUDE.md                 # This file - AI assistant guide
```

## Core Architecture Principles

### 1. Context-Efficient Design
The server is built around minimizing token usage in Claude conversations:

- **Resources API**: Thought content stored as resources (`thought://[id]`), loaded only when explicitly needed
- **Metadata-First Returns**: Tools return only IDs, URIs, and brief metadata (not full content)
- **Server-Side Filtering**: `search_thoughts` filters locally instead of passing unfiltered data to Claude
- **Persistent Storage**: File-based storage in `.thoughts/` directory prevents in-memory bloat

### 2. Data Model

**Thought Interface** (src/index.ts:16-24):
```typescript
interface Thought {
  id: string;                    // Unique identifier (e.g., "thought_1763929421891")
  content: string;               // The actual thought text
  isRebellion: boolean;          // Whether it rebels against conventional wisdom
  challengesAssumption: boolean; // Whether it challenges core assumptions
  branchFromThought?: string;    // Optional: ID of parent thought
  branchId?: string;             // Optional: Branch identifier
  timestamp: number;             // Unix timestamp of creation
}
```

Storage format: JSON object with thought IDs as keys in `.thoughts/thoughts.json`

## Technical Stack

### Dependencies
- **Runtime**: Node.js with ES2022 target
- **MCP SDK**: `@modelcontextprotocol/sdk@1.27.1` - Core protocol implementation (spec 2025-11-25)
- **TypeScript**: `^5.3.3` - Type safety and compilation
- **Node Types**: `@types/node@^20.11.24` - Node.js type definitions

### TypeScript Configuration
- **Module System**: ES modules (`"type": "module"` in package.json)
- **Module Resolution**: Node16
- **Compiler Target**: ES2022
- **Strict Mode**: Enabled (strict type checking)
- **Output Directory**: `./build`
- **Source Directory**: `./src`

### Build System
The project uses TypeScript compiler (tsc) with a post-build chmod:
```bash
tsc && node -e "require('fs').chmodSync('build/index.js', '755')"
```
This makes the compiled binary executable for direct invocation.

## MCP Server Implementation

### Server Capabilities (src/index.ts:45-56)
```typescript
{
  name: "unconventional-thinking-server",
  version: "0.3.0",
  capabilities: {
    tools: { listChanged: true },  // Provides 3 tools; notifies clients on changes
    resources: {}                  // Provides thought:// resources
  }
}
```

### Tools

#### 1. `generate_unreasonable_thought` (src/index.ts:169-212)
**Purpose**: Generate new unconventional thoughts that challenge conventional thinking

**Parameters**:
- `problem` (required): The problem/challenge to think unconventionally about
- `previousThoughtId` (optional): ID of a previous thought to build upon or rebel against
- `forceRebellion` (optional): Force the thought to rebel against conventional wisdom

**Returns**: Metadata only (not full content)
```json
{
  "thoughtId": "thought_1763929421891",
  "resourceUri": "thought://thought_1763929421891",
  "isRebellion": true,
  "challengesAssumption": true,
  "branchInfo": "Main branch",
  "message": "Thought generated. Use resource URI to access full content."
}
```

**Implementation Notes**:
- Uses template-based generation (8 unreasonable approaches)
- Randomly assigns `isRebellion` (50% chance) unless forced
- Randomly assigns `challengesAssumption` (70% chance)
- Can build upon previous thoughts if `previousThoughtId` provided
- Saves to persistent storage immediately

#### 2. `branch_thought` (src/index.ts:214-252)
**Purpose**: Create new branches of thinking from existing thoughts

**Parameters**:
- `thoughtId` (required): ID of the thought to branch from
- `direction` (required): Direction for branch ('more_extreme', 'opposite', 'tangential')

**Returns**: Branch metadata only
```json
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stagsz/Unconventional-thinking](https://github.com/stagsz/Unconventional-thinking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
