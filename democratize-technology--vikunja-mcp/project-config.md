---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Development Commands

### Pre-Commit Requirements (ALL must pass)
```bash
npm run lint           # ESLint validation
npm run test:coverage  # Jest with 90%+ branches, 95%+ lines coverage requirement  
npm run typecheck      # TypeScript compilation check
```

### Development Workflow
```bash
npm run build          # TypeScript compilation to dist/
npm run dev            # Watch mode development server with tsx
npm run test:watch     # Jest in watch mode for TDD

# Single test execution
jest tests/tools/tasks.test.ts              # Specific test file
jest -t "should create task"                # Specific test case by pattern
jest tests/tools/tasks-filters.test.ts      # Test specific functionality
```

### Build and Release
```bash
npm run format         # Prettier formatting for src/ and tests/
npm run prepare        # Pre-publish build step
npm run version:patch  # Bump patch version
```

## Architecture Overview

### MCP Server Pattern
This is a **Model Context Protocol (MCP) server** that exposes Vikunja task management operations as tools for AI assistants. The architecture follows a modular design with dependency injection:

- **Entry Point**: `src/index.ts` - Initializes McpServer with stdio transport
- **Tool Registry**: `src/tools/index.ts` - Centralized registration with conditional loading
- **Client Factory**: `src/client.ts` - Session-aware Vikunja API client management
- **Auth Manager**: Centralized authentication with JWT/API token auto-detection

### Tool Design Pattern
Each Vikunja entity follows a consistent **subcommand-based pattern**:
```typescript
server.tool('vikunja_tasks', {
  subcommand: z.enum(['create', 'get', 'update', 'delete', 'list']),
  // ... Zod validation schema
}, async (args) => {
  // Route to specific operation handlers
})
```

### Critical Architecture Decisions (Post-Refactoring v0.2.0)

1. **Simplified Storage Architecture (90% Code Reduction)**
   - Eliminated over-engineered 33-file storage system (9,803 lines)
   - Replaced with `SimpleFilterStorage.ts` (393 lines) for essential functionality
   - Thread-safe operations with AsyncMutex
   - Session-isolated storage with automatic cleanup
   - Located in `src/storage/SimpleFilterStorage.ts`

2. **Zod-Based Filter System (850+ Lines Removed)**
   - Replaced custom tokenizer/parser/validator with secure Zod schemas
   - Enhanced security with DoS protection and input validation
   - Production-ready parsing with comprehensive error handling
   - Located in `src/utils/filters-zod.ts`
   - Backward compatible filter syntax with improved reliability

3. **Production-Ready Retry System (580+ Lines Replaced)**
   - Replaced custom retry logic with battle-tested opossum library
   - Circuit breaker with state sharing and automatic recovery
   - Configurable timeouts, error thresholds, and reset behavior
   - Enhanced error handling for network failures
   - Located in `src/utils/retry.ts` with opossum integration

4. **Hybrid Filtering System**
   - Intelligent server-side filtering with client-side fallback
   - Automatic detection of server filtering capabilities
   - Enhanced memory protection with V8-specific memory estimation
   - Located in `src/tools/tasks/index.ts` and `src/utils/filters.ts`

5. **Enhanced Memory Protection System**
   - V8-specific memory estimation algorithms with 93%+ test coverage
   - Risk-based analysis (Low/Medium/High) with conservative 2.5x safety margins
   - Comprehensive task object modeling including nested arrays and dynamic properties
   - Backward compatible with legacy systems while providing improved accuracy
   - Located in `src/utils/memory.ts` with integration in `src/tools/tasks/filtering/FilterValidator.ts`

6. **Conditional Tool Registration**
   - Tools requiring JWT auth only registered when authenticated with JWT
   - API token authentication excludes `users` and `export` tools
   - Authentication type auto-detected by token format

7. **Session Management**
   - In-memory session persistence with client caching
   - Automatic client recreation on credential changes
   - No persistent storage - sessions reset on server restart

## Testing Philosophy & Requirements

### Strict Coverage Thresholds (ACHIEVED)
```json
"coverageThreshold": {
  "global": {
    "branches": 90,    // ✅ Current: 90%+
    "functions": 98,   // ✅ Current: 98.91% 
    "lines": 95,       // ✅ Current: 95%+
    "statements": 95   // ✅ Current: 95%+
  }
}
```

**Achievement**: All coverage thresholds have been met and are maintained through comprehensive test suites covering security scenarios, edge cases, and performance benchmarks.

### Defensive Programming Rule
**If code cannot be tested, it must be removed.** Every defensive pattern (like `|| ''` fallbacks) must have corresponding test cases that trigger those code paths.

Example pattern:
```typescript
// This defensive code MUST be testable
const message = error.message.toLowerCase() || '';
// Test MUST mock scenarios where error.message is undefined
```

### Test Organization
```
tests/
├── tools/           # Mirror src/tools structure exactly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [democratize-technology/vikunja-mcp](https://github.com/democratize-technology/vikunja-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
