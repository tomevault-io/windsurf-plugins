---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Monorepo Structure

This is a monorepo containing multiple packages. For package-specific guidance, refer to the individual CLAUDE.md files:

- **Main CLI Package**: @apps/ccusage/CLAUDE.md - Core ccusage CLI tool and library
- **Codex CLI Package**: @apps/codex/CLAUDE.md - OpenAI Codex usage tracking CLI
- **OpenCode CLI Package**: @apps/opencode/CLAUDE.md - OpenCode usage tracking CLI
- **MCP Server Package**: @apps/mcp/CLAUDE.md - MCP server implementation for ccusage data
- **Documentation**: @docs/CLAUDE.md - VitePress-based documentation website

Each package has its own development commands, dependencies, and specific guidelines. Always check the relevant package's CLAUDE.md when working within that package directory.

### Apps Are Bundled

All projects under `apps/` ship as bundled CLIs/binaries. Treat their runtime dependencies as bundled assets: list everything in each app's `devDependencies` (never `dependencies`) so the bundler owns the runtime payload.

## Development Commands

**Testing and Quality:**

- `pnpm run test` - Run all tests (using vitest via pnpm, watch mode disabled)
- Lint code using ESLint MCP server (available via Claude Code tools)
- `pnpm run format` - Format code with ESLint (writes changes)
- `pnpm typecheck` - Type check with TypeScript

**Build and Release:**

- `pnpm run build` - Build distribution files with tsdown
- `pnpm run release` - Full release workflow (lint + typecheck + test + build + version bump)

**Development Usage:**

- `pnpm run start daily` - Show daily usage report
- `pnpm run start monthly` - Show monthly usage report
- `pnpm run start session` - Show session-based usage report
- `pnpm run start blocks` - Show 5-hour billing blocks usage report
- `pnpm run start statusline` - Show compact status line (Beta)
- `pnpm run start daily --json` - Show daily usage report in JSON format
- `pnpm run start monthly --json` - Show monthly usage report in JSON format
- `pnpm run start session --json` - Show session usage report in JSON format
- `pnpm run start blocks --json` - Show blocks usage report in JSON format
- `pnpm run start daily --mode <mode>` - Control cost calculation mode (auto/calculate/display)
- `pnpm run start monthly --mode <mode>` - Control cost calculation mode (auto/calculate/display)
- `pnpm run start session --mode <mode>` - Control cost calculation mode (auto/calculate/display)
- `pnpm run start blocks --mode <mode>` - Control cost calculation mode (auto/calculate/display)
- `pnpm run start blocks --active` - Show only active block with projections
- `pnpm run start blocks --recent` - Show blocks from last 3 days (including active)
- `pnpm run start blocks --token-limit <limit>` - Token limit for quota warnings (number or "max")
- `node ./src/index.ts` - Direct execution for development

**MCP Server Usage:** (now provided by the `@ccusage/mcp` package)

- `pnpm dlx @ccusage/mcp@latest -- --help` - Show available options
- `pnpm dlx @ccusage/mcp@latest -- --type http --port 8080` - Start HTTP transport

**Cost Calculation Modes:**

- `auto` (default) - Use pre-calculated costUSD when available, otherwise calculate from tokens
- `calculate` - Always calculate costs from token counts using model pricing, ignore costUSD
- `display` - Always use pre-calculated costUSD values, show 0 for missing costs

**Environment Variables:**

- `LOG_LEVEL` - Control logging verbosity (0=silent, 1=warn, 2=log, 3=info, 4=debug, 5=trace)
  - Example: `LOG_LEVEL=0 pnpm run start daily` for silent output
  - Useful for debugging or suppressing non-critical output

**Multiple Claude Data Directories:**

This tool supports multiple Claude data directories to handle different Claude Code installations:

- **Default Behavior**: Automatically searches both `~/.config/claude/projects/` (new default) and `~/.claude/projects/` (old default)
- **Environment Variable**: Set `CLAUDE_CONFIG_DIR` to specify custom path(s)
  - Single path: `export CLAUDE_CONFIG_DIR="/path/to/claude"`
  - Multiple paths: `export CLAUDE_CONFIG_DIR="/path/to/claude1,/path/to/claude2"`
- **Data Aggregation**: Usage data from all valid directories is automatically combined
- **Backward Compatibility**: Existing configurations continue to work without changes

This addresses the breaking change in Claude Code where logs moved from `~/.claude` to `~/.config/claude`.

## Architecture Overview

This is a CLI tool that analyzes Claude Code usage data from local JSONL files stored in Claude data directories (supports both `~/.claude/projects/` and `~/.config/claude/projects/`). The architecture follows a clear separation of concerns:

**Core Data Flow:**

1. **Data Loading** (`data-loader.ts`) - Parses JSONL files from multiple Claude data directories, including pre-calculated costs
2. **Token Aggregation** (`calculate-cost.ts`) - Utility functions for aggregating token counts and costs
3. **Command Execution** (`commands/`) - CLI subcommands that orchestrate data loading and presentation
4. **CLI Entry** (`index.ts`) - Gunshi-based CLI setup with subcommand routing

**Output Formats:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryoppippi/ccusage](https://github.com/ryoppippi/ccusage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
