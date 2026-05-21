---
trigger: always_on
description: This file provides guidance to any AI coding agent (Claude Code, Cursor, etc.) working with this repository.
---

# Gaunt Sloth Assistant Internal Development Guidelines

This file provides guidance to any AI coding agent (Claude Code, Cursor, etc.) working with this repository.

## Technologies Used

- NodeJS 22 (LTS)
- Vitest 3 for tests
- Typescript 5
- LangChain and LangGraph 0.3

Please refer to package.json to check exact versions

## Core Development Principles

Vendor and system abstractions and wrappers should be used in most cases.

### Imports

Project uses import alias with `#src/*.js` pointing to `src/` and after build resolving to generated `dist/`.
Please abstain from using relative imports, only use them when no other choices are available
(currently the only exception is entry point cli.js)

### Architecture and Flow

- Make sure proper separation of LangChain components (LLMs, chains, agents, tools)
- Check for clear data flow between components
- Ensure proper state management in LangGraph workflows
- Validate error handling and fallback mechanisms

### Security

- Make sure API key handling and environment variables
- Make sure no personal data is present in code
- **Make sure that API keys are NOT accidentally included into diff.**
- Check for proper input sanitization
- Verify output validation and sanitization

### Output

Use [consoleUtils.ts](src/consoleUtils.ts) to output to users.
Do not use console.log directly.

### System

Use [systemUtils.ts](src/systemUtils.ts) to access system variables and functions such as
process.env, process.stdout, etc.

### LLM

Use [llmUtils.ts](src/llmUtils.ts) to access LLM.

### Middleware

Starting with v1.0.0, Gaunt Sloth uses LangChain middleware pattern instead of hooks.

Middleware provides hooks to intercept and control agent execution at critical points:
- `beforeModel`: Called before model invocation
- `afterModel`: Called after model response
- `beforeAgent`: Called before agent initialization
- `afterAgent`: Called after agent completion
- `wrapModelCall`: Wrap model calls with full control
- `wrapToolCall`: Wrap tool calls with full control

**Predefined Middleware:**
- `anthropic-prompt-caching`: Reduces API costs by caching prompts (Anthropic only)
- `summarization`: Condenses conversation history when approaching token limits

**Configuration:**
- Middleware is configured in the `middleware` array in config
- JSON configs support predefined middleware (string or config object)
- JS configs support both predefined and custom middleware objects

**Implementation:**
- Middleware registry is in [src/middleware/registry.ts](src/middleware/registry.ts)
- Middleware types are in [src/middleware/types.ts](src/middleware/types.ts)
- Provider-specific middleware can be auto-injected via `postProcessJsonConfig()` in preset files

## Tool Use

Precedence for your tool use:
1. Your built-in tools (e.g. Read, Edit, Write, Glob, Grep, etc.)
2. Bash commands that are documented in this file and in README.md
3. Other bash commands

**Examples of what to avoid:**
- ❌ `cat file.txt` → ✅ Use Read tool
- ❌ `grep pattern file.txt` → ✅ Use Grep tool
- ❌ `echo content > file.txt` → ✅ Use Write tool
- ❌ `find . -name "*.js"` → ✅ Use Glob tool

Abstain from using bash commands when you already have a built-in tool,
every time you use a bash command that is not in allow-list, it needs approval and slows down the process.

## Integration tests

Running all integration tests (takes ~10 minutes):

```bash
npm run it vertexai
```

Command accepts another argument which is a partial file name to filter tests,

for example `npm run it vertexai review` will run all tests that contain `review` in the file name.

Faster integration tests have `simple` suffix, which allows running a subset of tests quickly,
this also helps with less intelligent models:

```bash
npm run it vertexai simple
```

Run multiple integration test patterns:
```bash
npm run it vertexai prCommand reviewCommand
```

### Building and Testing

```bash
# Build the project
npm run build

# Run tests
npm test

# Run linting
npm run lint

# Auto-fix simple lint issues
npm run lint-n-fix

# Format code
npm run format

# Install globally for development
npm install -g ./
```

## Release Notes

Release notes are stored in `release-notes/` and follow a consistent format.

### Writing Release Notes

When creating release notes for a new version:

1. **File naming**: Use the pattern `v{major}_{minor}_{patch}.md` (e.g., `v1_1_0.md`)
2. **Title format**: `# v{major}.{minor}.{patch} {Brief Description}`
3. **Style**: Keep language dry and factual, not excited or marketing-oriented

### Structure

Release notes should include relevant sections:

- **New Features**: Major functionality additions
- **Potentially Breaking Changes**: Changes that might require user action
- **Bug Fixes**: Resolved issues
- **Improvements**: Refactoring, performance, architecture improvements
- **Maintenance**: Dependency updates, minor fixes

### Guidelines

- Focus on user-facing changes and impacts
- Omit internal implementation details like specific test counts or documentation updates unless relevant
- Use concrete examples where helpful
- For breaking changes, explain what users need to do
- Reference examples: `v1_0_0.md`, `v1_0_2.md`, `v1_0_4.md`, `v1_0_5.md`

### Example Structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Galvanized-Pukeko/gaunt-sloth-assistant](https://github.com/Galvanized-Pukeko/gaunt-sloth-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
