---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Build and Development
```bash
npm run build          # Compile TypeScript and make executable
npm run watch          # Watch mode for development
npm run prepare        # Pre-publish build (runs automatically)
```

### Testing
```bash
npm test               # Run all tests with Jest
npm run test:coverage  # Run tests with coverage report
npm test -- --testNamePattern="pattern"  # Run specific test by name pattern
npm test -- test/taskService.test.ts     # Run specific test file
```

### Debugging and Inspection
```bash
npm run inspector      # Launch MCP inspector at http://localhost:6274
node build/index.js    # Run server directly for testing
```

### Version Management
```bash
npm run update-version    # Update version across all config files
npm version patch|minor|major  # Standard npm version bump (auto-syncs)
```

## Architecture Overview

### Core Design Pattern
This is an **MCP (Model Context Protocol) server** that provides AI agents with project management and memory tools. The system uses **in-memory storage only** - no files are persisted to disk.

### Service-Oriented Architecture
```
src/
├── core/MemoryPickleCore.ts     # Main business logic orchestration
├── handlers/RequestHandlers.ts  # MCP protocol request handling
├── services/                    # Domain services
│   ├── InMemoryStore.ts        # Transaction-safe memory storage
│   ├── ProjectService.ts       # Project management logic
│   ├── TaskService.ts          # Task management logic
│   └── MemoryService.ts        # Context/memory management
├── tools/index.ts              # 12 MCP tools with enhanced prompts
└── types/schemas.ts            # Zod validation schemas
```

### Key Architectural Decisions

1. **In-Memory Only Storage**: All data exists only during the session. No file persistence eliminates concurrency issues and simplifies deployment.

2. **Transaction Safety**: InMemoryStore uses snapshot-based transactions to ensure data integrity without file locking complexity.

3. **Service Separation**: Clear boundaries between projects, tasks, and memories with dedicated services for each domain.

4. **MCP Protocol Compliance**: Strict adherence to MCP standards with proper error handling and clean text output.

### Data Flow
1. MCP client calls tool → RequestHandlers 
2. RequestHandlers validates input → MemoryPickleCore methods
3. MemoryPickleCore coordinates services → InMemoryStore transactions
4. Results flow back with clean text formatting

## Tool Architecture

The system provides **12 essential MCP tools** organized in clean categories:

**Read Tools:**
- `recall_state` - Get current project context with tasks, memories, and stats in one call
- `list_tasks` - List and filter tasks with pagination
- `list_projects` - List projects with completion stats  
- `get_task` - Get detailed info for a single task including subtasks and notes

**Write Tools:**
- `create_project` - Create new project and set as current
- `update_project` - Update project name, description, or status
- `set_current_project` - Switch active project context
- `create_task` - Create task in current or specified project
- `update_task` - Update task progress, completion, notes, or blockers

**Memory Tools:**
- `remember_this` - Store important information linked to current project/task

**Session Management:**
- `export_session` - Export session data as markdown or JSON for permanent storage
- `generate_handoff_summary` - Generate session summary for handoff between sessions

### Tool Design Principles
- **Action-oriented descriptions** with clear usage triggers
- **Auto-detection patterns** for priority and context
- **Proactive behavior hints** for AI agents
- **Single responsibility** per tool

## Session Memory Model

### Memory Lifecycle
- Data created at session start
- All tools share the same in-memory database
- Data destroyed when session ends
- **Handoff summaries** provide continuity between sessions

### Session Tracking
The core tracks comprehensive session activity:
- Tasks created/updated/completed
- Memories stored
- Project switches
- Tool usage patterns
- Key decisions made

## Development Practices

### Type Safety
- All data validated with **Zod schemas** at runtime
- TypeScript strict mode enforced
- Comprehensive error handling with specific error types

### Testing Strategy
- **16 comprehensive test files** covering all services and edge cases
- Test utilities in `test/utils/testHelpers.ts`
- Integration tests for MCP protocol compliance
- Coverage reporting available

### Error Handling
- Defensive programming throughout
- Clean error messages for MCP clients
- Graceful degradation when data not found
- No console.error output to prevent MCP stdio interference

## Version Management

The project uses **centralized version management**:
- Single source of truth in `package.json`
- `src/utils/version.ts` reads version dynamically
- `scripts/update-version.mjs` syncs configuration files
- Use `npm version patch` for automated version bumps

## Clean Text Output Philosophy

The system uses **clean text only** (no emojis) for:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Justar96/memory-pickle](https://github.com/Justar96/memory-pickle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
