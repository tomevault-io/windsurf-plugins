---
trigger: always_on
description: **A unified management interface for Claude Code configurations**
---

# Claude Code Config - VS Code Extension

**A unified management interface for Claude Code configurations**

This project is a VS Code extension that provides centralized management for Claude Code settings including memories (CLAUDE.md files), slash commands, skills, sub-agents, permissions, and hooks.

---

## Project Overview

### Purpose

Claude Code Config solves the problem of scattered configuration files by creating a single command center for managing all Claude Code settings. Instead of navigating between `~/.claude/` and `.claude/` directories, users can visualize, create, edit, and organize everything from a VS Code sidebar.

### Key Capabilities

- **Unified View**: All Claude Code configurations in one tree view interface
- **Scope Management**: Move configs between global (~/.claude/) and project (.claude/) with one click
- **Visual Hook Builder**: Create automation hooks without touching JSON
- **Folder Organization**: Group commands and sub-agents logically
- **Live Sync**: Auto-refresh when configuration files change
- **Color-Coded Agents**: Sub-agents display in configured colors

---

## Tech Stack

### Core Technologies

- **Language**: TypeScript (ES2022, strict mode enabled)
- **Build Tool**: esbuild (fast bundler with watch mode)
- **Runtime**: Node.js 20+
- **Platform**: VS Code Extension API 1.85.0+
- **Module Format**: CommonJS (required for VS Code extensions)

### Key Dependencies

- `chart.js ^4.4.1` - For analytics dashboard features
- `vscode` (external) - VS Code Extension API (never bundled)

### Development Tools

- ESLint with TypeScript parser
- TypeScript 5.3.2 compiler
- VS Code Extension Test framework
- esbuild for fast incremental builds

---

## Architecture

### Directory Structure

```
src/
├── extension.ts              # Main entry point, activation, command registration
├── core/
│   ├── types.ts             # All TypeScript interfaces and type definitions
│   └── constants.ts         # Shared constants, IDs, paths, and configs
├── providers/
│   └── claudeTreeDataProvider.ts  # All 8 tree view provider implementations
├── services/
│   ├── fileDiscoveryService.ts    # Discovers Claude config files across scopes
│   ├── fileOperationsService.ts   # CRUD operations for files and folders
│   ├── hooksService.ts            # Hooks JSON manipulation and management
│   ├── permissionsService.ts      # Permission rule parsing and discovery
│   └── mcpService.ts              # MCP server configuration discovery
└── utils/
    ├── yamlParser.ts        # YAML frontmatter extraction from markdown
    └── markdownParser.ts    # Markdown heading and section parsing

resources/
└── templates/               # File templates for quick creation
    ├── CLAUDE.md.template
    ├── SKILL.md.template
    └── command.md.template
```

### Key Architectural Patterns

#### 1. Service Layer Pattern

Business logic is separated into specialized services:

- **FileDiscoveryService**: Recursively scans and categorizes Claude config files
- **FileOperationsService**: Handles all file/folder CRUD operations
- **HooksService**: Manages hook JSON structures and validation
- **PermissionsService**: Parses permission rules from settings files
- **McpService**: Discovers and manages MCP server configurations

#### 2. Provider Pattern

VS Code TreeDataProviders for the 8 sidebar sections:

- MemoriesTreeProvider (CLAUDE.md files with sections)
- CommandsTreeProvider (slash command .md files)
- SkillsTreeProvider (SKILL.md in folders)
- SubAgentsTreeProvider (agent .md files with colors)
- PermissionsTreeProvider (permission rules visualization)
- HooksTreeProvider (hook automation organized by event)
- DocumentationTreeProvider (links to Claude Code docs)

#### 3. Command Pattern

21+ registered VS Code commands handle all user actions:

- File operations: open, rename, delete, move, copy path
- Creation: memory, command, skill, sub-agent, folder, hook
- Navigation: reveal in finder, go to specific line
- Hooks: create, edit, delete, duplicate, copy JSON

#### 4. Type Safety First

Comprehensive TypeScript interfaces in `core/types.ts`:

- ClaudeFile, ClaudeTreeItem, ClaudeScope
- Hook types, matchers, and event configurations
- Permission rules and patterns
- Service method signatures

---

## Code Style & Conventions

### Formatting

- **Indentation**: 2 spaces (no tabs)
- **Line Length**: 100 characters maximum
- **Quotes**: Single quotes for strings
- **Semicolons**: Always use semicolons
- **Trailing Commas**: Use in multi-line structures

### Naming Conventions

- **Variables**: camelCase (`fileDiscoveryService`, `treeItems`)
- **Functions**: camelCase (`createHook`, `discoverFiles`)
- **Classes**: PascalCase (`FileDiscoveryService`, `HooksService`)
- **Interfaces**: PascalCase (`ClaudeFile`, `HookConfiguration`)
- **Constants**: SCREAMING_SNAKE_CASE (`CLAUDE_GLOBAL_PATH`)
- **Files**: kebab-case (`file-discovery-service.ts`)

### Import Order

1. External dependencies (vscode, path, fs)
2. Internal core modules (types, constants)
3. Services
4. Utilities
5. Type-only imports (use `import type`)

Example:

```typescript
import * as vscode from "vscode";
import * as path from "path";
import * as fs from "fs/promises";

import { ClaudeFile, ClaudeScope } from "./core/types";

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drewipson/claude-code-config](https://github.com/drewipson/claude-code-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
