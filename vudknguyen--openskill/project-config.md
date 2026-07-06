---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Reference for AI Agents

- **Entry point**: `src/cli/index.ts`
- **Add new commands**: Create file in `src/cli/`, register in `index.ts`
- **Add new agents**: Implement `Agent` interface in `src/agents/`, register in `src/agents/index.ts`
- **Tests**: `src/__tests__/*.test.ts` (run with `npm test`)
- **Full architecture docs**: [docs/architecture.md](docs/architecture.md)

## Project Overview

OpenSkill (`osk`) is a lightweight, agent-agnostic CLI for managing AI coding agent skills. Supports Claude Code, Antigravity, OpenAI Codex, Cursor, Gemini CLI, GitHub Copilot, OpenCode, Windsurf, Kiro, Warp, OpenClaw, and the generic Agents spec.

## Build Commands

```bash
npm install          # Install dependencies
npm run build        # Compile TypeScript to dist/
npm run dev          # Watch mode for development
npm test             # Run tests with vitest
```

## Running the CLI

```bash
node dist/cli/index.js <command>   # Run locally
osk <command>                      # After npm link
```

## Architecture

```
src/
├── cli/           # Command handlers (commander.js)
│   ├── index.ts   # Entry point
│   ├── install.ts # Install with interactive agent/skill selection
│   └── ...        # list, uninstall, update, search, browse, repo, config, etc.
├── core/          # Business logic
│   ├── config.ts  # JSON config (~/.openskill/config.json)
│   ├── git.ts     # Git operations via Node child_process
│   ├── skill.ts   # Skill parsing/validation
│   ├── registry.ts # Git repository cache operations
│   └── manifest.ts # Installed skills tracking (~/.openskill/manifest.json)
├── agents/        # Agent adapters (strategy pattern)
│   ├── types.ts   # Agent interface
│   └── *.ts       # claude, antigravity, codex, cursor, gemini, copilot, opencode, windsurf, kiro, warp, openclaw, agents
└── utils/
    ├── logger.ts  # ANSI colors (no deps)
    ├── prompt.ts  # Interactive prompts (@inquirer/prompts)
    ├── markdown.ts # Frontmatter parsing (gray-matter)
    └── fs.ts      # File helpers
```

## Dependencies

Minimal by design - 4 runtime dependencies:

- `commander` - CLI framework
- `gray-matter` - YAML frontmatter parsing
- `@inquirer/prompts` - Interactive prompts (select, checkbox, confirm, input, search)
- `tar` - Tar archive creation/extraction for marketplace packages

All other functionality uses Node.js built-ins.

## CLI Commands

| Command                    | Alias | Description                                 |
| -------------------------- | ----- | ------------------------------------------- |
| `install <source> [skill]` | `i`   | Install skills from repository or by name   |
| `uninstall <skill>`        | `rm`  | Remove installed skills                     |
| `list [name]`              | `ls`  | List installed skills                       |
| `update`                   | `up`  | Check and apply skill updates               |
| `search <query>`           | `s`   | Search for skills                           |
| `browse`                   | `b`   | Browse available skills                     |
| `login`                    |       | Authenticate with the marketplace           |
| `logout`                   |       | Log out and revoke token                    |
| `whoami`                   |       | Show current login status                   |
| `push [path]`              |       | Push skill to marketplace as draft          |
| `publish <slug>`           |       | Make a pushed skill public                  |
| `audit [path]`             |       | Run security audit on a skill               |
| `repo add <source>`        |       | Add a skill repository                      |
| `repo ls`                  |       | List configured repositories                |
| `repo rm <name>`           |       | Remove a repository                         |
| `repo sync [name]`         |       | Sync repositories (fetch latest skills)     |
| `repo info <name>`         |       | Show repository details                     |
| `config`                   |       | Show/modify configuration                   |
| `init [name]`              |       | Create new skill template                   |
| `validate [path]`          |       | Validate skill format                       |
| `convert <source>`         |       | Convert between skill formats               |
| `which <skill>`            |       | Show skill installation path                |
| `man [command]`            |       | Show detailed help                          |
| `completion [shell]`       |       | Generate shell completion (bash, zsh, fish) |
| `version`                  | `v`   | Show version information                    |

## Pre-commit checks

Before creating any commit, always run all four checks:

```bash
npm run lint              # ESLint — must pass with 0 errors
npm run format:check      # Prettier formatting — must pass
npm run build             # tsc typecheck — must pass with 0 errors
npm test -- --run         # Vitest — all tests must pass
```

Or use the single shortcut that runs all of them in sequence:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vudknguyen/openskill](https://github.com/vudknguyen/openskill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
