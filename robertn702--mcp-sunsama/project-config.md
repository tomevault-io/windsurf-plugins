---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Development
bun run dev                 # Run server with .env file
bun run typecheck          # TypeScript type checking
bun run typecheck:watch    # Watch mode type checking
bun run inspect            # MCP Inspector for debugging

# Testing
bun test                   # Run unit tests only
bun test:unit              # Run unit tests only (alias)
bun test:integration       # Run integration tests (requires credentials)
bun test:all               # Run all tests
bun test:watch             # Watch mode for unit tests

# Build and Distribution
bun run build              # Compile TypeScript to dist/
bun run prepublishOnly     # Run build before publish

# Version Management (Changeset)
bun run changeset          # Create new changeset
bun run version            # Apply changesets and update version
bun run release            # Build and publish to npm
```

## Contribution Workflow

**IMPORTANT**: When making changes that affect package users, always create a changeset.

### When to Create a Changeset
- ✅ New user-facing features or tools
- ✅ Bug fixes that affect npm package users
- ✅ API changes or breaking changes
- ✅ Dependency updates that change behavior

### When NOT to Create a Changeset
- ❌ Infrastructure/deployment changes (CI/CD, Docker, Smithery config)
- ❌ Internal refactoring with no behavior changes
- ❌ Documentation updates
- ❌ Development tooling changes

### Standard Workflow
1. Create a branch: `git checkout -b {type}/{short-name}`
2. Make changes and test: `bun test && bun run typecheck`
3. **Create a changeset**: `bun run changeset` (if changes affect users)
4. Commit with conventional format: `git commit -m "fix: description"`
5. Push and create PR

See `CONTRIBUTING.md` for full details.

## Architecture Overview

### Dual Transport MCP Server
This server supports two transport modes with different authentication strategies:

**Stdio Transport** (default):
- Single global SunsamaClient authenticated at startup
- Uses `SUNSAMA_EMAIL`/`SUNSAMA_PASSWORD` environment variables
- Session maintained for entire server lifetime

**HTTP Stream Transport**:
- Per-request authentication via HTTP Basic Auth
- Session-isolated SunsamaClient instances
- Credentials provided in Authorization header

Transport selection via `TRANSPORT_MODE` environment variable ("stdio" | "http").

### Session Management Architecture
For HTTP transport, the server implements dual-layer session caching:

**Client Cache Layer** (`utils/client-resolver.ts`):
- In-memory Map caching authenticated SunsamaClient instances
- SHA-256 hashed credential keys for security
- Automatic cache invalidation on authentication failure

**Session Manager Layer** (`session/session-manager.ts`):
- Manages session lifecycle with configurable TTL
- Tracks session metadata (createdAt, lastAccessedAt)
- Automatic cleanup of expired sessions
- Transport reference management for proper cleanup

### Client Resolution Pattern
`utils/client-resolver.ts` abstracts transport differences:
- **Stdio**: Returns singleton client from global authentication
- **HTTP**: Extracts client from session data (authenticated per request)
- Throws standardized errors for unauthenticated requests

### Response Optimization Strategy
Two-tier optimization for large datasets:

1. **Task Filtering** (`utils/task-filters.ts`): Filter by completion status before processing
2. **Task Trimming** (`utils/task-trimmer.ts`): Remove non-essential fields to reduce payload by 60-80%

Always apply filtering before trimming for efficiency.

### Enhanced Pagination Pattern
The `get-archived-tasks` tool implements smart pagination:

- **Limit+1 Pattern**: Fetches `requestedLimit + 1` to determine if more results exist
- **Pagination Metadata**: Returns `hasMore` flag, `nextOffset`, and count information
- **LLM Context**: Provides clear guidance for AI assistants on whether to continue fetching
- **Response Format**: TSV data with pagination header for optimal processing

### Schema Architecture
All tools use Zod schemas from `schemas.ts`:
- Type-safe parameter validation
- Automatic TypeScript inference
- Comprehensive parameter documentation
- Union types for completion filters
- **Important**: Avoid using `.refine()` on schemas - it transforms `ZodObject` into `ZodEffects` which the MCP SDK cannot parse (results in empty `properties`). Handle complex validation (e.g., XOR between fields) in the tool's `execute` function instead.
- Example: `update-task-notes` requires either `html` OR `markdown`, validated at runtime
- Discriminated unions for task integrations (GitHub, Gmail)

### Integration Support
The `create-task` tool supports external integrations:
- **GitHub Integration**: Link tasks to GitHub issues or pull requests
- **Gmail Integration**: Link tasks to Gmail emails
- Uses discriminated union pattern for type-safe integration handling
- Integration parameter is optional to maintain backward compatibility

## Key Patterns

### Tool Structure
Modern tool pattern using shared utilities and parameter destructuring:
```typescript
// Old pattern (before refactoring)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robertn702/mcp-sunsama](https://github.com/robertn702/mcp-sunsama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
