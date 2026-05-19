---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Model Context Protocol (MCP) server that provides SvelteKit and Tailwind CSS documentation and code snippets. Built with TypeScript using ES modules, it features security-hardened file operations, input validation, and caching.

**Key Technology Stack:**
- MCP SDK v1.17+ with Smithery deployment
- TypeScript 5.3+ with strict mode
- Node.js 20+ (ES modules required)
- Zod for schema validation

## Build and Development Commands

### Standard Development
```bash
npm run build          # Build using Smithery CLI (outputs to dist/)
npm run dev            # Development mode with hot reload
npm run watch          # TypeScript watch mode
npm run inspector      # Launch MCP inspector for testing
```

### Content Management
```bash
npm run update-content # Scrape and update docs from SvelteKit/Tailwind sites
```

### Maintenance
```bash
npm run security-audit # Run security audit (moderate level)
npm run outdated-check # Check for outdated dependencies
```

## Architecture

### Module System
- **ES Modules Only**: Uses `"type": "module"` in package.json
- All imports must use `.js` extensions even for `.ts` files (TypeScript requirement)
- No CommonJS `require()` - use `import` statements

### Core Architecture Pattern

The server follows a modular security-first design:

```
src/index.ts                    # Smithery HTTP transport server factory
  ↓ creates
McpServer instance
  ↓ registers tools with
SecureFileService              # Handles all file I/O with caching
  ↓ uses
validateToolInput()            # Input validation (security.ts)
sanitizeAndValidatePath()      # Path traversal protection
ErrorHandler                   # Safe error formatting
```

### Key Architectural Principles

1. **Smithery Integration**: The server uses `createServer()` factory pattern that accepts config from smithery.yaml and returns an MCP server instance

2. **Path Resolution**: All paths use `process.cwd()` as base (not `__dirname`) for Smithery compatibility - the working directory is the project root when deployed

3. **Security Layers**:
   - Input validation with regex patterns (alphanumeric + `-_.` only)
   - Path sanitization to prevent `../` traversal
   - Base path boundary checking
   - File size limits (1MB default)
   - Safe error messages (no internal details exposed)

4. **Caching Strategy**: LRU cache with 5-minute timeout in SecureFileService - cache keys are full resolved file paths

5. **Error Handling Hierarchy**:
   ```
   McpError (from SDK)
     ← thrown by tools
     ← converted by ErrorHandler.formatSafeErrorMessage()
     ← catches: ValidationErrors, FileSystemErrors, UnknownErrors
   ```

### Content Organization

```
content/
├── docs/
│   ├── sveltekit/     # SvelteKit docs (.md files)
│   └── tailwind/      # Tailwind CSS docs (.md files)
└── snippets/
    └── [category]/    # Component directories
        └── *.svelte   # Svelte component files
```

File naming: Use kebab-case with alphanumeric characters, hyphens, underscores, and dots only.

## MCP Tools Available

The server exposes 11 tools with full parameter descriptions and annotations:

**Legacy Documentation Tools** (limited coverage):
- `get_sveltekit_doc` - Read SvelteKit documentation by topic (~8% coverage)
- `get_tailwind_info` - Read Tailwind CSS documentation by query (~4% coverage)

**Full Documentation Tools** (complete coverage, large context):
- `get_svelte_full_docs` - Complete Svelte/SvelteKit docs (~320k tokens, 100% coverage)
- `get_tailwind_full_docs` - Complete Tailwind CSS docs (~730k tokens, 100% coverage)
  - **WARNING**: These return very large responses. Use search tools for smaller contexts.

**Search Tools** (recommended for most use cases):
- `search_svelte_docs` - Search within complete Svelte documentation
- `search_tailwind_docs` - Search within complete Tailwind documentation

**Component Snippet Tools**:
- `get_component_snippet` - Read Svelte component snippet by category/name
- `list_snippet_categories` - List available component categories (17 categories)
- `list_snippets_in_category` - List snippets within a specific category

**List Tools**:
- `list_sveltekit_topics` - List available SvelteKit doc files
- `list_tailwind_info_topics` - List available Tailwind doc files

All tools include:
- Comprehensive parameter descriptions
- Read-only, non-destructive, idempotent annotations
- Audit logging and structured error handling
- Input validation and security checks

## Code Patterns and Conventions

### TypeScript Strictness
- No `any` types allowed
- All parameters must have explicit types
- Use `as const` for literal type assertions
- Return types are explicit on public methods

### Security Validation Pattern
Every tool that accepts user input follows this pattern:
```typescript
validateToolInput(toolName, request);  // Throws McpError if invalid
const content = await fileService.readSecureFile(basePath, userInput, extension);
```

### Audit Logging Pattern
All tool executions log:
```typescript
createAuditLog('info', 'tool_request', { tool: 'name', timestamp, argsProvided });
// ... operation ...

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CaullenOmdahl/Tailwind-Svelte-Assistant](https://github.com/CaullenOmdahl/Tailwind-Svelte-Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
