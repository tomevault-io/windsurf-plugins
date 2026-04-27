---
trigger: always_on
description: `@krodak/clickup-cli` (`cup`) - a ClickUp CLI for AI agents and humans. TypeScript, ESM-only, Node 22+. Three output modes: interactive tables with task picker in TTY, Markdown when piped (optimized for AI context windows), JSON with `--json`. The binary is `cup` - the previous `cu` name was retired to avoid conflict with the Unix `cu(1)` utility.
---

# AGENTS.md

## Project Overview

`@krodak/clickup-cli` (`cup`) - a ClickUp CLI for AI agents and humans. TypeScript, ESM-only, Node 22+. Three output modes: interactive tables with task picker in TTY, Markdown when piped (optimized for AI context windows), JSON with `--json`. The binary is `cup` - the previous `cu` name was retired to avoid conflict with the Unix `cu(1)` utility.

## Project Skills

This repo includes project-level agent skills in `.agents/skills/`:

| Skill                     | When to use                                                                                                                                  |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **releasing-clickup-cli** | Releasing a new version (npm, Homebrew, GitHub Release, skill sync). Follow this skill exactly - it prevents version/docs/metadata mistakes. |
| **testing-clickup-cli**   | Running tests, adding test coverage, debugging failures. Documents unit test patterns, e2e workspace fixtures, and the metadata sync test.   |

Use these skills for releasing and testing instead of the sections below - they have full step-by-step instructions.

## External Skills

- **typescript-pro** - for all TypeScript work. The project uses strict mode, `verbatimModuleSyntax`, `noUncheckedIndexedAccess`, and `typescript-eslint` recommendedTypeChecked rules.
- **cli-developer** - for CLI design, argument parsing, interactive prompts, and shell completions. The project uses Commander for CLI framework, @inquirer/prompts for interactive UI, and chalk for colors.

## Tech Stack

| Tool              | Purpose                                                        |
| ----------------- | -------------------------------------------------------------- |
| TypeScript        | strict, ES2022 target, NodeNext modules                        |
| tsup              | Build - single ESM bundle to `dist/index.js`                   |
| Vitest            | Unit tests (`tests/unit/`) and e2e tests (`tests/e2e/`)        |
| ESLint 10         | Flat config with typescript-eslint recommendedTypeChecked      |
| Prettier          | No semicolons, single quotes, trailing commas, 100 print width |
| Commander         | CLI framework                                                  |
| @inquirer/prompts | Interactive terminal UI                                        |
| chalk             | Terminal colors                                                |

## Project Structure

```
src/
  index.ts          # CLI entry point (Commander setup)
  api.ts            # ClickUp API client (ClickUpClient class + types)
  config.ts         # Config loading (~/.config/cup/config.json)
  output.ts         # TTY detection, table formatting, shouldOutputJson
  interactive.ts    # Task pickers, TTY detail views (chalk)
  markdown.ts       # Markdown detail views (piped output)
  date.ts           # Date formatting helpers
  commands/         # One file per command
tests/
  unit/             # Mirrors src/ structure, *.test.ts
  e2e/              # Integration tests, *.e2e.ts (requires .env.test)
docs/
  commands.md       # Full command reference with examples and flags
  api-coverage.md   # API coverage matrix with status indicators
skills/
  clickup-cli/      # Agent skill file shipped with npm package
.agents/skills/
  using-clickup-cli/ # Agent skill (canonical location for npx skills add)
  releasing-clickup-cli/ # Internal: release process (metadata.internal: true)
  testing-clickup-cli/   # Internal: test guide (metadata.internal: true)
.claude-plugin/
  plugin.json       # Claude Code plugin manifest
```

## Development Commands

```bash
npm install         # Install dependencies
npm test            # Unit tests (runs build first via globalSetup)
npm run test:e2e    # E2E tests (requires CLICKUP_API_TOKEN in .env.test)
npm run build       # tsup -> dist/
npm run dev         # Run from source via tsx
npm run typecheck   # tsc --noEmit
npm run lint        # ESLint
npm run lint:fix    # ESLint with auto-fix
npm run format      # Prettier write
npm run format:check # Prettier check
```

## Code Conventions

- ESM only - all imports use `.js` extensions (`import { foo } from './bar.js'`)
- No inline comments - code should be self-documenting through naming
- Unused variables prefixed with `_` (enforced by ESLint)
- No floating promises (enforced by ESLint `no-floating-promises: error`)
- Use `type` imports for type-only imports (`import type { Foo }`)
- Every command lives in its own file under `src/commands/`
- Every command file has a corresponding test file under `tests/unit/commands/`

## Adding a New Command

1. Create `src/commands/<name>.ts` with the command logic
2. Register the command in `src/index.ts` using Commander
3. Add to `src/commands/metadata.ts` (completion test will fail otherwise)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krodak/clickup-cli](https://github.com/krodak/clickup-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
