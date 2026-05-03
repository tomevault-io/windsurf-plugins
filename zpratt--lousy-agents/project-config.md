---
trigger: always_on
description: **Mandatory**: Always read [](./context/project.context.md) for full project context before contributing.
---


# Project Overview

**Mandatory**: Always read [](./context/project.context.md) for full project context before contributing.

## Shared Instruction Architecture

This repo provides instructions for both GitHub Copilot and Claude Code. The detailed domain instructions in `.github/instructions/` are the shared source-of-truth referenced by both systems.

`CLAUDE.md` uses `@path/to/file` syntax (e.g., `@.github/instructions/test.instructions.md`) to import those shared files. This is Claude Code's native file reference mechanism — **not** a broken markdown link. Do not suggest converting `@path/to/file` references in `CLAUDE.md` to markdown links.

## Commands

Mise manages all tools and Node versions. If you haven't activated mise in your shell, run `mise activate` once or prefix commands with `mise exec --`. During development, use file-scoped commands for faster feedback, and run the full validation suite (`mise run ci && npm run build`) before commits.

```bash
# One-time shell setup (or add to ~/.zshrc)
eval "$(mise activate zsh)"

# Core commands
mise run test            # Run tests (vitest)
npm run build            # Production build
mise run format-check    # Lint check
mise run format-fix      # Auto-fix lint/format

# File-scoped (faster feedback)
npx biome check path/to/file.ts
npm test path/to/file.test.ts

# Validation suite (run before commits)
mise run ci && npm run build

# Linting tasks
mise run actionlint      # Validate GitHub Actions (actionlint)
mise run yamllint        # Validate YAML (yamllint)
mise run lint            # Run all linting tools in parallel

# Other
npm audit                # Security check
npm install              # Install deps (updates package-lock.json)
```

**Note**: In GitHub Actions, `jdx/mise-action` automatically activates mise and makes all tools available in PATH. No additional setup needed in CI.

This repository is an npm workspace monorepo. Run `npm install` once at the root to install all workspace dependencies. The root `npm run build` command builds the publishable packages: `packages/cli`, `packages/mcp`, and `packages/agent-shell`.

## Workflow: TDD Required

Follow this exact sequence for ALL code changes. Work in small increments — make one change at a time and validate before proceeding.

1. **Research**: Search codebase for existing patterns, components, utilities. Use Context7 MCP tools for library/API documentation.
2. **Write failing test**: Create test describing desired behavior
3. **Verify failure**: Run `mise run test` — confirm clear failure message
4. **Implement minimal code**: Write just enough to pass
5. **Verify pass**: Run `mise run test` — confirm pass
6. **Refactor**: Clean up, remove duplication, keep tests green
7. **Validate**: `mise run ci && npm run build`

Task is NOT complete until all validation passes.

## Tech Stack

- **Framework**: CLI using c12 for configuration management and citty for terminal interactions
  - When choosing additional libraries, prefer ones that integrate well with c12/citty (from [UnJS ecosystem](https://unjs.io/))
- **Language**: TypeScript (strict mode)
- **Validation**: Zod for runtime validation of external data
- **Testing**: Vitest (never Jest), MSW for HTTP mocking, Chance.js for test fixtures
- **Linting**: Biome (never ESLint/Prettier separately)
- **Logging**: Consola with JSON format and child loggers
- **HTTP**: fetch API only
- **Architecture**: Clean Architecture principles

## Project Structure

```
.github/           GitHub Actions workflows
packages/
  core/            Shared domain entities, use cases, gateways, and formatters
  cli/             Published CLI package and reference scaffold templates
  mcp/             Published MCP server package
  action/          Private GitHub Action package
  agent-shell/     Published npm script-shell telemetry package
scripts/           Build, deploy, and test scripts
.nvmrc             Node.js version (latest LTS)
```

## Code Style

```typescript
import { z } from 'zod';

// Define schema for runtime validation
const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
});

type User = z.infer<typeof UserSchema>;

// ✅ Good - small, typed, single purpose, descriptive names, runtime validation
async function fetchUserById(userId: string): Promise<User> {
  if (!userId) {
    throw new Error('User ID required');
  }

  const response = await fetch(`/api/users/${userId}`);

  if (!response.ok) {
    throw new Error(`Failed to fetch user: ${response.status}`);
  }

  const data: unknown = await response.json();
  return UserSchema.parse(data);
}

// ❌ Bad - untyped, type assertion on external data, no validation, multiple responsibilities, impure (side effects: global state mutation)
async function doStuff(x) {
  console.log('fetching');
  globalState.loading = true;
  const response = await fetch('/api/users/' + x);
  return response.json() as User;
}
```

**Rules:**
- Always use TypeScript type hints
- Use descriptive names for variables, functions, and modules
- Functions must be small and have single responsibility
- Avoid god functions and classes — break into smaller, focused units
- Avoid repetitive code — extract reusable functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zpratt/lousy-agents](https://github.com/zpratt/lousy-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
