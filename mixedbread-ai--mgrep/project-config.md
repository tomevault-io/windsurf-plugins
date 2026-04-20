---
trigger: always_on
description: > This file provides context and guidelines for AI coding agents working on the mgrep codebase. It follows the [AGENT.md specification](https://github.com/agentmd/agent.md) and incorporates best practices from Anthropic, OpenAI, and Google.
---

# AGENTS.md

> This file provides context and guidelines for AI coding agents working on the mgrep codebase. It follows the [AGENT.md specification](https://github.com/agentmd/agent.md) and incorporates best practices from Anthropic, OpenAI, and Google.

## Project Overview

**mgrep** is a semantic search CLI tool that brings natural language understanding to code search. It's built with TypeScript and designed to work seamlessly with AI coding agents.

### Core Purpose

- Replace pattern-based grep with semantic, natural-language search
- Index and search code, PDFs, images, and text files
- Integrate with coding agents (Claude Code, Codex, OpenCode, Factory Droid)
- Provide web search capabilities alongside local file search

### Tech Stack

- **Runtime**: Node.js (ESM modules)
- **Language**: TypeScript 5.x with strict mode
- **Package Manager**: pnpm
- **Linting/Formatting**: Biome
- **Testing**: bats (Bash Automated Testing System)
- **API**: Mixedbread SDK for semantic search

## Project Structure

```
mgrep/
├── src/
│   ├── index.ts              # CLI entry point, command registration
│   ├── commands/             # CLI command implementations
│   │   ├── login.ts          # Authentication flow
│   │   ├── logout.ts         # Session cleanup
│   │   ├── search.ts         # Core search functionality
│   │   ├── switch-org.ts     # Organization switching
│   │   ├── watch.ts          # File watching and indexing
│   │   └── watch_mcp.ts      # MCP (Model Context Protocol) watch
│   ├── install/              # Agent integration installers
│   │   ├── claude-code.ts    # Claude Code integration
│   │   ├── codex.ts          # Codex integration
│   │   ├── droid.ts          # Factory Droid integration
│   │   └── opencode.ts       # OpenCode integration
│   └── lib/                  # Shared utilities and core logic
│       ├── auth.ts           # Authentication utilities
│       ├── config.ts         # Configuration management
│       ├── context.ts        # Execution context
│       ├── file.ts           # File operations
│       ├── git.ts            # Git integration
│       ├── logger.ts         # Logging utilities
│       ├── organizations.ts  # Organization management
│       ├── store.ts          # Mixedbread store operations
│       ├── sync-helpers.ts   # Synchronization utilities
│       ├── token.ts          # Token management
│       ├── utils.ts          # General utilities
│       └── warning.ts        # Warning display utilities
├── test/
│   ├── test.bats             # Integration tests
│   └── assets/               # Test fixtures
├── plugins/                  # Plugin definitions
├── guides/                   # Documentation guides
└── .claude-plugin/           # Claude Code plugin config
```

## Build & Development Commands

```bash
# Install dependencies
pnpm install

# Build the project
pnpm build

# Development build + run
pnpm dev

# Run tests
pnpm test

# Format and lint code
pnpm format          # Auto-fix issues
pnpm format:check    # Check only (CI)
pnpm lint            # Lint check

# Type checking
pnpm typecheck
```

## Code Style & Conventions

### TypeScript Standards

- **Strict mode enabled** - All code must pass `tsc --strict`
- **Explicit return types** - All exported functions must declare return types
- **Prefer interfaces** - Use interfaces over types for public APIs
- **No `any`** - Use `unknown` or generics instead

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Classes | PascalCase | `FileProcessor` |
| Functions/Variables | camelCase | `processFile` |
| Constants | UPPER_SNAKE_CASE | `MAX_FILE_SIZE` |
| Files | kebab-case | `sync-helpers.ts` |

### Import Organization

```typescript
// 1. Node.js built-ins
import * as fs from "node:fs";
import * as path from "node:path";

// 2. Third-party packages
import { program } from "commander";
import chalk from "chalk";

// 3. Internal modules (absolute paths)
import { auth } from "./lib/auth.js";

// 4. Local files (relative paths)
import { search } from "./commands/search.js";
```

### Documentation

- **JSDoc for all exports** - Include parameter descriptions and return types
- **Minimal inline comments** - Code should be self-explanatory
- **Comment "why" not "what"** - Only explain non-obvious decisions

## Testing Guidelines

### Running Tests

```bash
# Run all tests (excludes long-running tests)
pnpm test

# Run specific test
bats test/test.bats --filter "test name pattern"

# Run including long-running tests
bats test/test.bats
```

### Writing Tests

- Tests use **bats** (Bash Automated Testing System)
- Test files go in `test/` directory
- Use descriptive test names: `@test "search returns results for valid query"`
- Tag long-running tests with `# bats test_tags=long-running`

### Test Requirements

When making changes:
1. **Bug fixes** - Add a regression test
2. **New features** - Add integration tests covering happy path and edge cases
3. **Refactors** - Ensure existing tests pass

## Git Workflow

### Commit Messages

Follow conventional commits: `type(scope): description`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mixedbread-ai/mgrep](https://github.com/mixedbread-ai/mgrep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
