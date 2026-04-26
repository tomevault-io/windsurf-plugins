---
trigger: always_on
description: This is the Relentless codebase - a universal AI agent orchestrator that works with multiple AI coding agents (Claude Code, Amp, OpenCode, Codex, Droid, Gemini).
---

# Relentless - Universal AI Agent Orchestrator

## Overview

This is the Relentless codebase - a universal AI agent orchestrator that works with multiple AI coding agents (Claude Code, Amp, OpenCode, Codex, Droid, Gemini).

**Recent Major Updates (January 2026):**
- **v0.2.0**: Multi-agent skills installation - all 6 agents now get skills via `relentless init`
- Skills architecture: commands are thin wrappers, logic in `.claude/skills/`
- Agent-specific folders: `.amp/skills/`, `.opencode/skill/`, `.codex/skills/`, `.factory/skills/`, `.gemini/GEMINI.md`
- Full support: Claude, Amp, OpenCode | Manual workflow: Codex, Droid, Gemini

## Codebase Patterns

### Directory Structure

- `bin/` - CLI entry point (relentless.ts)
- `src/` - Core TypeScript implementation
  - `agents/` - Agent adapters for each AI coding agent
  - `config/` - Configuration schema and loading
  - `prd/` - PRD parsing and validation
  - `execution/` - Orchestration loop and routing
  - `init/` - Project initialization scaffolder
- `.claude/skills/` - Skills for Claude Code (also copied to other agents)
- `.claude/commands/` - Command wrappers that load skills

### Key Concepts

1. **Agent Adapters**: Each AI agent has an adapter implementing `AgentAdapter` interface
2. **Skills Architecture**: Commands load skills which contain the actual logic and templates
3. **PRD Format**: User stories in `prd.json` with `passes: true/false` status (generated from `tasks.md`)
4. **Completion Signal**: `<promise>COMPLETE</promise>` indicates all stories done
5. **Progress Log**: `progress.txt` accumulates learnings across iterations
6. **Multi-Tier Agent Support**: Full skills support (Claude/Amp/OpenCode), Extensions (Gemini), Manual (Droid/Codex)
7. **Learning System**: Capture learnings from completed features to improve future runs via `/relentless.learn`

### Development Commands

```bash
# Install locally for development
bun install

# Run the orchestrator during development
bun run bin/relentless.ts run --feature <name>

# Type check
bun run typecheck

# Lint
bun run lint
```

### Testing Locally

```bash
# Test init command
mkdir /tmp/test && cd /tmp/test
bun run /path/to/relentless/bin/relentless.ts init

# Test with a simple PRD
cd /path/to/relentless
bun run bin/relentless.ts run --feature <name>
```

### Using the Global Binary

```bash
# Install globally
bun install -g .

# Run from anywhere
relentless init
relentless run --feature <name>
```

### Code Style

- TypeScript with strict mode
- Bun as runtime (no Node.js)
- Zod for schema validation
- Commander for CLI parsing

## Testing

Relentless follows Test-Driven Development (TDD). All new features must have tests written BEFORE implementation.

### Running Tests

```bash
# Run all tests
bun test

# Run tests with coverage report
bun test --coverage

# Run specific test file
bun test tests/queue/parser.test.ts

# Watch mode for development
bun test --watch
```

### Test File Structure

```
tests/
├── helpers/           # Shared test utilities
│   └── index.ts       # Temp dirs, file helpers, fixtures
├── queue/             # Queue module tests
│   └── parser.test.ts # Parser unit tests
├── integration/       # Integration tests (future)
└── e2e/              # End-to-end tests (future)
```

### Test File Naming Convention

- Unit tests: `*.test.ts` (placed alongside tests/ or near source)
- Integration tests: `tests/integration/*.test.ts`
- E2E tests: `tests/e2e/*.test.ts`

### TDD Workflow (Red-Green-Refactor)

1. **Red Phase**: Write failing tests that define expected behavior
   ```typescript
   it("should parse valid queue line", () => {
     const result = parseQueueLine("2026-01-13T10:00:00.000Z | Fix bug");
     expect(result).not.toBeNull();
     expect(result?.content).toBe("Fix bug");
   });
   ```

2. **Green Phase**: Write minimum code to pass tests
   ```typescript
   export function parseQueueLine(line: string): QueueItem | null {
     const match = line.match(/^(.+?) \| (.+)$/);
     if (!match) return null;
     return { timestamp: match[1], content: match[2] };
   }
   ```

3. **Refactor Phase**: Clean up while keeping tests green
   - Extract constants
   - Improve naming
   - Remove duplication

### Test Helpers

The `tests/helpers/` module provides utilities for testing:

```typescript
import {
  createTempDir,
  createTestFile,
  readTestFile,
  fixtures,
  mockTimestamp,
} from "../helpers";

// Create isolated temp directory
const { path, cleanup } = await createTempDir();

// Create test files
await createTestFile(path, ".queue.txt", "content");

// Generate fixtures
fixtures.validQueueLine("Fix the bug");  // "2026-01-13T10:00:00.000Z | Fix the bug"
fixtures.queueCommand("PAUSE");           // "[PAUSE]"
fixtures.queueCommand("SKIP", "US-003");  // "[SKIP US-003]"

// Always cleanup
await cleanup();
```

### Coverage Requirements

- Aim for >80% code coverage on new features
- Critical paths must have 100% coverage
- Edge cases and error conditions must be tested

### Important Files

- `bin/relentless.ts` - Main CLI entry point
- `src/agents/` - Agent adapters for each supported agent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArvorCo/Relentless](https://github.com/ArvorCo/Relentless) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
