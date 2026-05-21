---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## Overview

Polka Codes is an AI-powered coding assistant framework built with TypeScript and Bun. It provides a CLI tool that helps developers with task planning, code generation, debugging, and git workflows through natural language interactions and a multi-agent system.

## Development Commands

### Building
```bash
bun build                 # Build all packages
bun clean                 # Remove build artifacts
```

### Testing
```bash
bun test                  # Run tests
bun test:coverage         # Coverage report (see docs/TESTING_GUIDELINES.md)
```

### Type Checking and Linting
```bash
bun typecheck            # Type check only
bun lint                 # Check linting (biome)
bun fix                  # Auto-fix issues
bun check                # Type check + lint
```

### Running the CLI
```bash
bun cli <command>        # Run CLI in dev mode
bun pr                   # Create PRs
bun commit               # Create commits
```

## Project Architecture

### Monorepo Structure

- **`packages/core`**: AI services, workflow engine, agents, tools
- **`packages/cli`**: CLI interface
- **`packages/cli-shared`**: Shared utilities
- **`packages/github`**: GitHub integration
- **`packages/runner`**: Agent runner service

### Workflow System

**Core Workflow** (`packages/core/src/workflow/workflow.ts`):
- `WorkflowFn<TInput, TOutput, TTools>` - Core workflow type
- `BaseWorkflowContext<TTools>` - Provides `step`, `logger`, `tools`
- `step` function - Named execution units with retry and caching
- `ToolRegistry` - Type-safe tool registry

**Agent Workflow** (`packages/core/src/workflow/agent.workflow.ts`):
- Core agentic execution loop
- Tool calling and message flow
- Retry support and result caching
- JSON output schemas for structured responses

**Dynamic Workflows** (`packages/core/src/workflow/dynamic.ts`):
- YAML-based workflow definitions
- AI-agent-executed steps
- Sub-workflow calls via `runWorkflow`
- State management across steps

### Tool System

**Tool Definition** (`packages/core/src/tool.ts`):
- `ToolInfo`: Name, description, Zod schema
- `FullToolInfo`: Adds handler implementation
- `ToolResponse`: Reply, Exit, or Error
- Parameters MUST use `z.object`

**Core Tools** (`packages/core/src/tools/`):
- File operations: `readFile`, `writeToFile`, `replaceInFile`, `removeFile`
- Search: `search`, `searchFiles`, `listFiles`
- Execution: `executeCommand`
- AI: `askFollowupQuestion`, `fetchUrl`
- Memory: `readMemory`, `updateMemory`, `listMemoryTopics`
- Todo: `getTodoItem`, `updateTodoItem`, `listTodoItems`
- Skills: `loadSkill`, `listSkills`

### Agent Skills System

**Skill Storage** (priority order):
1. `.claude/skills/` - Project skills (git-tracked)
2. `~/.claude/skills/` - Personal skills
3. `node_modules/@polka-codes/skill-*/` - Plugin skills

**SKILL.md Format**:
```yaml
---
name: react-component-generator
description: Generate React components
allowed-tools: [readFile, writeToFile]
---

# React Component Generator
Instructions here...
```

**Skill Commands**:
- `bun cli skills list` - List skills
- `bun cli skills validate <name>` - Validate skill
- `bun cli skills create <name>` - Create skill scaffold

### Configuration

**File**: `.polkacodes.yml` in project root

**Key Fields**:
- `providers`: AI provider configs (API keys, models)
- `scripts`: Test, format, check commands
- `rules`: Custom instructions for agents
- `excludeFiles`: Glob patterns to exclude
- `toolFormat`: "native" or "polka-codes"

### AI Provider Abstraction

**Supported**: DeepSeek, Anthropic, Ollama, OpenRouter, Google Vertex

**Model Resolution**:
1. Command-specific config
2. Provider default
3. Global default

## Code Conventions

### General
- Use `#methodName` / `#fieldName` for private members
- Use `bun` as package manager
- Use `bun:test` for testing (NOT jest, mocha, or vi)
- DO NOT mock in unit tests - use real implementations
- `biome` for linting (NOT prettier or eslint)
- ToolInfo parameters MUST be `z.object`
- Use `.nullish()` instead of `.optional()` in Zod
- Avoid global variables
- Avoid `typeof` - use strong typing

### Testing

**Comprehensive Guidelines**: See `docs/TESTING_GUIDELINES.md` for:
- Core principles and best practices
- When to write tests (and when not to)
- Test structure and organization
- Common patterns for tools, workflows, async operations
- Coverage requirements

**Snapshot Testing**: Use for tool outputs and structured data
```typescript
expect(result).toMatchSnapshot()
```

**Error Testing**: Use `.rejects.toThrow()` (NOT try-catch)
```typescript
await expect(someTool({ input: null })).rejects.toThrow('Invalid input')
```

**Coverage**:
```bash
bun test:coverage          # Text report
bun test:coverage:lcov     # LCov for CI
bun test:coverage:html     # HTML report
```

**Important**: DO NOT create summary documents. Existing docs in `plans/` and `docs/` are sufficient.

### Error Handling

**Principles**:
- Prefer explicit error types
- Let errors propagate to appropriate handler
- Use `unknown` not `any` in catch blocks
- Only catch errors you can handle

**When to Catch**:

1. **Graceful Degradation** (log and continue):
```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [polka-codes/polka-codes](https://github.com/polka-codes/polka-codes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
