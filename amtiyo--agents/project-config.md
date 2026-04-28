---
trigger: always_on
description: Instructions for AI coding agents working on the `@agents-dev/cli` project.
---

# AGENTS.md

Instructions for AI coding agents working on the `@agents-dev/cli` project.

## Project Overview

**@agents-dev/cli** is a CLI tool that provides a practical standard layer for multi-LLM development. It solves the configuration fragmentation problem by syncing MCP servers, skills, and instructions across multiple AI coding tools (Codex, Claude Code, Gemini CLI, Cursor, Copilot, Antigravity) from a single source of truth.

**Key Principle:** One `.agents/` folder syncs to all tools. Add an MCP server once, it appears everywhere.

## Tech Stack

- **Language:** TypeScript (strict mode)
- **Runtime:** Node.js 20+
- **Build:** tsc (TypeScript compiler)
- **Tests:** Vitest
- **CLI Framework:** Commander.js
- **Prompts:** @clack/prompts
- **Package Manager:** npm

## Project Structure

```
agents/
├── src/
│   ├── cli.ts                    # CLI entry point
│   ├── commands/                 # Command implementations
│   │   ├── start.ts             # Setup wizard
│   │   ├── sync.ts              # Config sync
│   │   ├── mcp-add.ts           # Add MCP server
│   │   └── ...
│   ├── core/                     # Core logic
│   │   ├── sync.ts              # Sync orchestration
│   │   ├── mcp.ts               # MCP management
│   │   ├── renderers.ts         # Tool-specific config generators
│   │   ├── trust.ts             # Codex trust management
│   │   ├── ui.ts                # CLI output helpers (colors, spinners)
│   │   └── ...
│   ├── integrations/             # Tool integrations
│   │   ├── codex.ts
│   │   ├── claude.ts
│   │   ├── cursor.ts
│   │   └── ...
│   └── types.ts                  # TypeScript types
├── tests/                        # Test files
│   ├── *.test.ts                # Unit tests
│   └── *.integration.test.ts    # Integration tests
├── templates/                    # Scaffold templates
│   └── agents/                  # .agents/ directory templates
├── docs/                         # Public documentation
└── bin/agents                    # CLI wrapper script
```

## Development Workflow

### Setup

```bash
npm install
npm run build
npm link
```

### Development Commands

```bash
npm run dev -- <command>          # Run CLI in dev mode
npm run build                     # Compile TypeScript
npm test                          # Run all tests
npm test tests/sync.test.ts       # Run specific test
npm test -- --watch               # Watch mode
npm run lint                      # Run ESLint
```

### Testing

- **Unit tests:** Test individual functions in isolation
- **Integration tests:** Test full command flows with temp directories
- Use `mkdtemp` for temporary test directories
- Always clean up in `afterEach` hooks
- Mock external CLI calls when appropriate

### Code Style

- **TypeScript strict mode** is enabled
- Follow existing patterns and conventions
- Use descriptive variable names
- Add JSDoc comments for exported functions
- Keep functions focused (single responsibility)
- Prefer pure functions where possible

## Key Concepts

### 1. Project Structure (`core/project.ts`)

Projects have:
- `.agents/agents.json` — MCP servers, shared config
- `.agents/local.json` — Secrets (gitignored)
- `.agents/skills/` — Reusable workflows
- `.agents/generated/` — Auto-generated files (gitignored)
- `AGENTS.md` — Instructions for all tools

### 2. Sync Process (`core/sync.ts`)

1. Read `.agents/agents.json` and `.agents/local.json`
2. Merge configs (local overrides shared)
3. Generate tool-specific configs via renderers
4. Write atomically (temp file + rename)
5. Acquire sync lock to prevent race conditions

### 3. Renderers (`core/renderers.ts`)

Each tool has a renderer that converts `.agents/agents.json` to tool-specific format:
- **Codex:** TOML (`.codex/config.toml`)
- **Claude:** JSON (`.claude/mcp.json`)
- **Gemini:** JSON (`.gemini/settings.json`)
- **Cursor:** JSON (`.cursor/mcp.json`)
- **Copilot:** JSON (`.vscode/mcp.json`)
- **Antigravity:** JSON (`.antigravity/mcp.json`)

### 4. MCP Server Management (`core/mcp.ts`)

MCP servers have:
- **Transport:** `stdio` (command-based) or `http`/`sse` (URL-based)
- **Config:** command, args, env, headers
- **Secrets:** Stored in `.agents/local.json`
- **Validation:** Keys must be shell-safe (env) or HTTP token format (headers)

### 5. Trust Management (`core/trust.ts`)

Codex requires explicit trust per project. We:
- Check trust state via `codex trust list`
- Set trust via `codex trust set --path <project>`
- Store trust in Codex global config

### 6. File Safety (`core/fs.ts`)

Critical operations use atomic writes:
- Write to temp file
- Rename to target (atomic on most filesystems)
- Prevents data corruption if process crashes

## Common Tasks

### Adding a New Command

1. Create `src/commands/your-command.ts`
2. Export async function matching command signature
3. Register in `src/cli.ts` with Commander
4. Add tests in `tests/your-command.test.ts`
5. Update README.md command reference
6. Update CHANGELOG.md

### Adding a New Tool Integration

1. Create `src/integrations/your-tool.ts`
2. Implement integration interface (id, name, paths)
3. Add renderer in `src/core/renderers.ts`
4. Register in `src/integrations/registry.ts`
5. Add sync logic in `src/core/sync.ts`
6. Add tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amtiYo/agents](https://github.com/amtiYo/agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
