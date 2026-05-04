---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands
- Build all packages: `pnpm build`
- Build specific package: `cd packages/<name> && pnpm build`
- Dev mode (watch): `pnpm dev` or `cd packages/<name> && pnpm dev`
- Type check: `pnpm typecheck`
- Lint: `pnpm lint`
- Test: `pnpm test`
- Clean: `pnpm clean`

## Project Structure

This is a pnpm monorepo containing Claude Code extensions:

```
claude-skills/
├── packages/          # MCP servers (TypeScript, builds to dist/)
│   ├── mcp-github-issues/
│   └── mcp-debug-cycle/
├── skills/           # Agent Skills (SKILL.md format)
│   └── code-review/
└── hooks/            # Claude Code hooks (Node.js scripts)
    ├── cost-monitor/
    ├── notify-complete/
    └── tdd-enforcement/
```

## Architecture

### MCP Servers (`packages/`)

TypeScript packages following MCP SDK patterns:

- **Entry**: `src/index.ts` exports `Server` from `@modelcontextprotocol/sdk/server/index.js`
- **Tools**: `src/tools/*.ts` export `Tool` definitions and implementation functions
- **Utils**: `src/utils/*.ts` for shared logic (e.g., `gh-cli.ts` wraps GitHub CLI)
- **Build**: TypeScript compiles `src/` → `dist/`, using `tsconfig.base.json` + package-level `tsconfig.json`
- **Execution**: `dist/index.js` is the MCP server entry point (runs via `node`)

Tool implementations follow this pattern:
```typescript
export const toolName: Tool = {
  name: "mcp_tool_name",
  description: "...",
  inputSchema: { /* Zod or JSON Schema */ }
}

export async function toolName(args: ArgsType): Promise<ToolResponse> {
  // Implementation
}
```

### Agent Skills (`skills/`)

Agent Skills Specification (agentskills.io) compliant format:

- **Structure**: Each skill is a directory with `SKILL.md` as the manifest
- **Frontmatter**: YAML with required fields `name` (lowercase-hyphen only) and `description`
- **Content**: Markdown body with skill instructions, using XML-style tags for structure:
  - `<essential_principles>` - Core philosophy and rules
  - `<intake>` - User-facing prompt for skill invocation
  - `<routing>` - Pattern matching to workflow files
  - `<reference_index>` - Links to reference docs
  - `<workflows_index>` - Available workflow files
- **Supporting files**:
  - `references/` - On-demand docs (e.g., checklists, common issues)
  - `workflows/` - Step-by-step procedures (e.g., review-diff.md)
  - `scripts/` - Executable code (if needed)
  - `assets/` - Templates, data files

### Hooks (`hooks/`)

Node.js scripts triggered by Claude Code lifecycle events:

- **Structure**: Each hook is a directory with `hooks.json` (config) + `.js` (implementation)
- **Config**: `hooks.json` defines matcher (e.g., `"PostToolUse"`, `"Stop"`, `"PreToolUse"`)
- **Implementation**: Node.js script receives stdin with event data, performs action
- **Critical**: Use absolute paths in `hooks.json` command field (cwd resets between invocations)
- **State**: No in-memory persistence - load/save from files (e.g., `~/.claude/usage.json`)

## Key Patterns

### MCP Tool Safety
- **Shell injection prevention**: Use `execFile` not `exec` (see `gh-cli.ts`)
- **Input validation**: Zod schemas in tool definitions
- **Error handling**: Return `{ content: [...], isError: true }` for errors

### Agent Skills Compatibility
- **Naming**: Skill names must match directory name, lowercase-hyphen only
- **Progressive disclosure**: Keep SKILL.md body <5000 tokens, use `references/` for detail
- **Cross-agent**: SKILL.md format works across different AI agents (Claude, GPT, etc.)

### TypeScript Configuration
- **Base config**: `tsconfig.base.json` has strict mode, ES2022 target, NodeNext modules
- **Package configs**: Extend base, set `outDir: "dist"` and `rootDir: "src"`
- **Imports**: Use `.js` extensions in TypeScript imports (ES modules requirement)

## Development Workflow

### Adding a New MCP Server
1. Create `packages/mcp-<name>/` directory
2. Add `package.json` with `"type": "module"` and `@modelcontextprotocol/sdk` dependency
3. Create `src/index.ts` with Server setup
4. Create `src/tools/*.ts` for each tool
5. Add `tsconfig.json` extending `../../tsconfig.base.json`
6. Build with `pnpm build`
7. Test by adding to `~/.claude/settings.json` mcpServers config

### Adding a New Skill
1. Create `skills/<skill-name>/` directory
2. Create `SKILL.md` with frontmatter (name, description) and markdown body
3. Optional: Add `references/`, `workflows/`, `scripts/` subdirectories
4. Validate naming: lowercase-hyphen only, no consecutive hyphens
5. Install with `cp -r skills/<skill-name> ~/.claude/skills/`

### Adding a New Hook
1. Create `hooks/<hook-name>/` directory
2. Create `hooks.json` with matcher and absolute path to script
3. Create Node.js script that reads stdin, performs action
4. Use absolute paths for file operations (cwd is unstable)
5. Install by merging `hooks.json` into `~/.claude/hooks.json`

## Cross-Agent Compatibility

Skills in this repo follow the Agent Skills Specification (agentskills.io):

- **Portable format**: SKILL.md works across Claude, GPT, Gemini, etc.
- **No lock-in**: Markdown + YAML frontmatter, no proprietary JSON

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasonkneen/agent-skills](https://github.com/jasonkneen/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
