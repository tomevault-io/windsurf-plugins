---
trigger: always_on
description: Please also reference the following documents as needed:
---

Please also reference the following documents as needed:

@.claude/memories/msw-handlers.md description: "MSW handler rules for integration test fidelity" globs: "**/tests/mocks/**/*"
## Development Guidelines

### Package Management

- **ALWAYS use pnpm** - never use npm
- Install dependencies with `pnpm add` or `pnpm add -D` to ensure latest versions
- Never manually edit `package.json` dependencies - use pnpm commands
- For CI/CD, use non-interactive flags: `pnpm install --frozen-lockfile`

### TypeScript Configuration

- Target: ES2022 with NodeNext module resolution
- Strict mode enabled
- Source maps for debugging
- Use `tsx` for development execution (already configured in dev scripts)

### Testing Approach

- Unit tests mirror source directory structure
- Use Vitest for testing framework (migrating from Mocha)
- Follow Test-Driven Development (TDD) practices
- For detailed testing guidelines, see `docs/testing-strategy.md` and the TDD agents

### Working with Test Infrastructure and Code

When modifying test infrastructure (MSW handlers, test utilities, mock data):

- **ALWAYS read** `.claude/agents/tdd-test-writer.md` FIRST for requirements and patterns
- This includes creating new handlers, updating existing ones, or adding mock utilities

When implementing or modifying code (whether making tests pass or any other changes):

- **ALWAYS read** `.claude/agents/test-driven-coder.md` FIRST for implementation patterns

### Environment Configuration

- Copy `.env.example` to `.env` for local development
- Required: API keys for providers (Dune, Birdeye, etc.)
- Chain configurations in environment variables
- **Node.js native environment variable loading** - we do NOT use the `dotenv` package
  - Node.js 20.6+ supports native `.env` file loading via the `--env-file` flag
  - Integration and e2e test scripts use `tsx --env-file=.env.test` to load test environment variables
  - See `package.json` scripts: `test:int`, `test:e2e`, and `test:record-mocks`
  - No need to manually call `dotenv/config` or import `dotenv`

### Docker Development

- `compose.local.yaml` for local development with Memgraph
- `compose-tests.yaml` for test environment
- `compose.yaml` for production deployment

### Managing Commands, Subagents, and Rules

**Source of truth**: `.rulesync/` directory - never edit generated `.claude/` or `.cursor/` files

**Workflow**:

1. Create/edit files in `.rulesync/{commands,subagents,rules}/`
2. Run `pnpm sync:rules` to generate to `.claude/` and `.cursor/`

**Frontmatter formats**:

Commands (`.rulesync/commands/*.md`):

```yaml
---
description: "Brief description"
targets: ["*"]
allowed-tools: ["Bash", "Read", "Write", "Edit", "Grep", "Glob", "Task"]
argument-hint: "(optional) argument hint"
---
```

Subagents (`.rulesync/subagents/*.md`):

```yaml
---
name: agent-name
targets: ["*"]
description: When to use this agent
claudecode:
  model: sonnet # or opus
  color: green
---
```

Rules (`.rulesync/rules/*.md`):

```yaml
---
root: true # for root.md only
targets: ["*"]
description: "Rule description"
globs: ["**/*"]
---
```

**Config**: `rulesync.jsonc` controls targets (cursor, claudecode) and features (rules, commands, subagents)

## Code Quality Standards

### General Best Practices

- Never use `--force` flags (e.g., `git push --force`) without explicit approval
- Don't wrap code in try/catch blocks only to add context - propagate errors directly
- Avoid trivial comments that merely restate the next line of code
- Never redefine existing interfaces - always import and reuse
- Never produce mocks instead of real implementations
- **NEVER use `any` type** - use proper types, `unknown`, or type assertions with `as`
- Never use `.passthrough()` with Zod schemas

### Refactoring and Breaking Changes

**CRITICAL: NEVER maintain backwards compatibility. This is an internal codebase, not a public library.**

When refactoring:

- ✅ Update ALL call sites immediately
- ✅ Make breaking changes directly
- ❌ NO compatibility aliases, re-exports, or type aliases (e.g., `type OldName = NewName`)
- ❌ NO deprecation warnings or transition periods
- ❌ NO keeping old names/paths alongside new ones

```typescript
// ❌ WRONG
export const newName = () => {
  /* ... */
};
export const oldName = newName; // NO!

// ✅ CORRECT - rename and update all call sites
export const newName = () => {
  /* ... */
};
```

### Schema Validation (Zod)

- Always validate external API inputs/outputs at application boundaries (adapters/clients in `src/`) using `zod`.
- Do not run schema validation in tests' MSW handlers or mock loaders; handlers must replay recorded responses unmodified.
- Optional: run mock drift/shape checks in CI or as a separate developer command, not during test runtime.
- Prefer a single `zod` major version across the workspace to avoid version conflicts.

### Decision Documentation

- Document significant architectural and implementation decisions in `/docs/rationales.md`
- **User approval required**: Always ask user before adding entries to rationales.md
- For detailed guidelines on what to document, see the Documentation Agent

### Pull Request Workflow

- **All changes must be merged via PR** - direct commits to main are not allowed
- Create a new branch and draft PR together before making changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EmberAGI/arbitrum-vibekit](https://github.com/EmberAGI/arbitrum-vibekit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
