---
trigger: always_on
description: **IMPORTANT:** Before implementing any feature, consult the specifications in `specs/README.md`.
---

# Wreckit Agent Guidelines

## Specifications

**IMPORTANT:** Before implementing any feature, consult the specifications in `specs/README.md`.

- **Assume NOT implemented.** Many specs describe planned features that may not yet exist in the codebase.
- **Check the codebase first.** Before concluding something is or isn't implemented, search the actual code. Specs describe intent; code describes reality.
- **Use specs as guidance.** When implementing a feature, follow the design patterns, types, and architecture defined in the relevant spec.
- **Spec index:** `specs/README.md` lists all specifications organized by phase.

## Commands

### Building & Testing

| Command | Does |
|---------|------|
| `bun build` | Build the CLI |
| `bun test` | Run all tests |
| `bun test src/__tests__/foo.test.ts` | Run single test file |
| `bun run typecheck` | Type check the codebase |
| `bun run lint` | Lint the codebase |

### CLI Commands

| Command | Does |
|---------|------|
| `wreckit` | Run all incomplete items (research → plan → implement → PR) |
| `wreckit next` | Run next incomplete item |
| `wreckit run <id>` | Run single item through all phases (id: `1`, `2`, or `001-slug`) |
| `wreckit ideas < FILE` | Ingest ideas (create idea items) |
| `wreckit learn` | Extract patterns and compile into reusable skills |
| `wreckit status` | List all items + state |
| `wreckit list` | List items (with optional `--state` filtering) |
| `wreckit show <id>` | Show item details |
| `wreckit init` | Initialize `.wreckit/` in repo |
| `wreckit doctor` | Validate items, fix broken state |
| `wreckit rollback <id>` | Rollback a direct-merge item to pre-merge state |
| `wreckit geneticist` | Recursive prompt self-optimization (evolution) |

### Phase Commands (debugging)

| Command | Transition |
|---------|------------|
| `wreckit research <id>` | idea → researched |
| `wreckit plan <id>` | researched → planned |
| `wreckit implement <id>` | planned → implementing |
| `wreckit pr <id>` | implementing → in_pr |
| `wreckit complete <id>` | in_pr → done |

### Flags

- `--verbose` — More logs
- `--quiet` — Errors only
- `--debug` — JSON output (ndjson)
- `--no-tui` — Disable UI (CI mode)
- `--dry-run` — Preview, don't execute
- `--force` — Regenerate artifacts
- `--cwd <path>` — Override working directory
- `--fix` — Auto-repair (doctor only)

## Meta-Agents

Wreckit includes specialized "meta-agents" that operate on the system itself rather than just executing tasks.

| Agent | Command | Purpose |
|-------|---------|---------|
| **Dreamer** | `wreckit dream` | **Ideation**: Scans the codebase for TODOs, gaps, and technical debt to autonomously generate new roadmap items. Acts as a product manager. |
| **Strategy** | `wreckit strategy` | **Navigator**: Analyzes the project state and roadmap to recommend the next best action. Acts as a technical lead. |
| **Learn** | `wreckit learn` | **Skills**: Extracts reusable patterns from completed items and compiles them into skills.json. Acts as a knowledge engineer. |
| **Geneticist** | `wreckit geneticist` | **Evolution**: Analyzes healing logs to identify recurrent failures and autonomously optimizes system prompts via PRs. Acts as an immune system. |

## State Flow

```
idea → researched → planned → implementing → in_pr → done
```

## Architecture

TypeScript CLI built with Bun. Key directories:

- `src/index.ts` — CLI entry, commands
- `src/domain/` — State machine, item indexing
- `src/commands/` — Phase handlers
- `src/agent/` — Agent subprocess and SDK integration
- `src/agent/mcp/` — MCP server for structured output
- `src/git/` — Git operations
- `src/fs/paths.ts` — Path helpers (items stored in `.wreckit/items/`)
- `specs/` — Feature specifications

## Config

`.wreckit/config.json`:
```json
{
  "schema_version": 1,
  "base_branch": "main",
  "branch_prefix": "wreckit/",
  "merge_mode": "pr",
  "agent": {
    "kind": "claude_sdk",
    "model": "claude-sonnet-4-20250514"
  },
  "max_iterations": 100,
  "timeout_seconds": 3600,
  "branch_cleanup": {"enabled": true, "delete_remote": true}
}
```

### Agent Kind Options

| Kind | Description |
|------|-------------|
| `claude_sdk` | Claude Agent SDK (default, recommended) |
| `amp_sdk` | Amp SDK (experimental) |
| `codex_sdk` | Codex SDK (experimental) |
| `opencode_sdk` | OpenCode SDK (experimental) |
| `process` | External CLI process |
| `rlm` | RLM Mode (ReAct Loop Mode) via @ax-llm/ax |

See [README.md](./README.md#agent-options) for configuration examples for each kind.

### Environment Variable Resolution

When using SDK mode (`claude_sdk`, `amp_sdk`, `codex_sdk`, or `opencode_sdk`), environment variables are merged from multiple sources with this precedence (highest first):

1. `.wreckit/config.local.json` `agent.env` (project-specific, gitignored)
2. `.wreckit/config.json` `agent.env` (project defaults)
3. `process.env` (shell environment)
4. `~/.claude/settings.json` `env` (Claude user settings)

Example `.wreckit/config.local.json` for custom API routing:
```json
{
  "agent": {
    "env": {
      "ANTHROPIC_BASE_URL": "https://api.z.ai/api/anthropic",
      "ANTHROPIC_AUTH_TOKEN": "your-token-here"
    }
  }
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikehostetler/wreckit](https://github.com/mikehostetler/wreckit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
