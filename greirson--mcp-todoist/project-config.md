---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

- `npm run build` - Compiles TypeScript to JavaScript in the dist/ directory
- `npm run prepare` - Runs build (used by npm automatically)
- `npm run watch` - Watches for TypeScript changes and rebuilds automatically
- `npm run lint` - Lints TypeScript files with ESLint
- `npm run lint:fix` - Auto-fixes linting issues
- `npm run format` - Formats code with Prettier
- `npm run format:check` - Checks code formatting
- `npm run test` - Runs Jest test suite
- `npm run test:watch` - Runs tests in watch mode
- `npm run test:coverage` - Runs tests with coverage report

## Architecture

This is an MCP (Model Context Protocol) server that integrates Claude with the Todoist API. The codebase has been modularized into a well-structured architecture.

### Key Components

- **MCP Server**: Uses `@modelcontextprotocol/sdk` for MCP protocol implementation
- **Todoist Integration**: Uses `@doist/todoist-api-typescript` client library
- **Transport**: Runs on stdio transport for communication with MCP clients

### Modular Architecture

The codebase follows a clean, domain-driven architecture with focused modules for improved maintainability:

#### Core Infrastructure

- **`src/index.ts`**: Main server entry point (121 lines) - delegates routing to router module
- **`src/router/`**: Tool routing module (extracted from index.ts):
  - `index.ts` - Route dispatcher function
  - `legacy-router.ts` - Legacy 60+ individual tool routing
  - `unified-router.ts` - Unified 19 tool routing
- **`src/types/`**: Modularized TypeScript type definitions (extracted from types.ts):
  - `task-types.ts`, `project-types.ts`, `bulk-types.ts`, `comment-types.ts`
  - `label-types.ts`, `api-types.ts`, `subtask-types.ts`, `reminder-types.ts`
  - `filter-types.ts`, `activity-types.ts`, `collaboration-types.ts`
  - `index.ts` - Re-exports all types
- **`src/type-guards/`**: Modularized runtime type validation (extracted from type-guards.ts):
  - `task-guards.ts`, `project-guards.ts`, `bulk-guards.ts`, `comment-label-guards.ts`
  - `subtask-guards.ts`, `reminder-guards.ts`, `filter-guards.ts`, `advanced-guards.ts`
  - `index.ts` - Re-exports all guards
- **`src/validation/`**: Modularized input validation (extracted from validation.ts):
  - `sanitization.ts` - XSS prevention and security patterns
  - `task-validation.ts`, `date-validation.ts`, `id-validation.ts`
  - `content-validation.ts`, `bulk-validation.ts`
  - `index.ts` - Re-exports all validators
- **`src/cache/`**: Modularized caching infrastructure (extracted from cache.ts):
  - `simple-cache.ts` - SimpleCache class with TTL
  - `cache-manager.ts` - CacheManager singleton
  - `index.ts` - Re-exports
- **`src/errors.ts`**: Custom error types with structured error handling

**Backwards Compatibility:** Original files (`src/types.ts`, `src/type-guards.ts`, `src/validation.ts`, `src/cache.ts`) are thin re-export files maintaining all existing import paths.

#### Modular Tool Organization

- **`src/tools/`**: Domain-specific MCP tool definitions (refactored from single 863-line file):
  - `task-tools.ts` - Task management tools (CREATE, READ, UPDATE, DELETE, COMPLETE, REOPEN + bulk operations with duration support)
  - `subtask-tools.ts` - Subtask management tools (hierarchical task operations)
  - `project-tools.ts` - Project, section, and collaborator management tools
  - `comment-tools.ts` - Comment creation and retrieval tools
  - `label-tools.ts` - Label CRUD and statistics tools
  - `filter-tools.ts` - Filter management tools (Sync API, requires Pro/Business plan)
  - `reminder-tools.ts` - Reminder management tools
  - `duplicate-tools.ts` - Duplicate detection and merging tools
  - `activity-tools.ts` - Activity log and audit trail tools
  - `backup-tools.ts` - Backup listing and download tools
  - `collaboration-tools.ts` - Workspace, invitation, and notification tools
  - `item-operations-tools.ts` - Task move, reorder, and close tools
  - `project-notes-tools.ts` - Project notes CRUD tools
  - `project-operations-tools.ts` - Project reorder and archive tools
  - `section-operations-tools.ts` - Section move, reorder, and archive tools
  - `shared-label-tools.ts` - Shared label management tools (Business)
  - `user-tools.ts` - User info and productivity stats tools
  - `test-tools.ts` - Testing and validation tools
  - `index.ts` - Centralized exports with backward compatibility

#### Business Logic Handlers

- **`src/handlers/`**: Domain-separated business logic:
  - **`src/handlers/task/`**: Modularized task handlers (extracted from task-handlers.ts):
    - `crud.ts` - Single task operations (create, get, update, delete, complete, reopen)
    - `bulk.ts` - Bulk operations and filterTasksByCriteria
    - `completed.ts` - Completed tasks retrieval via Sync API
    - `quick-add.ts` - Quick add with natural language parsing
    - `index.ts` - Re-exports all task handlers
  - `task-handlers.ts` - Re-exports from task/ for backwards compatibility
  - `subtask-handlers.ts` - Hierarchical task management and parent-child relationships

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greirson/mcp-todoist](https://github.com/greirson/mcp-todoist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
