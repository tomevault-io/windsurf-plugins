---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Buoy is a design drift detection tool for AI-generated code. It scans codebases to catch when developers (especially AI tools like Copilot/Claude) diverge from design system patterns before code ships.

## Common Commands

```bash
# Build all packages (required before running CLI)
pnpm build

# Build specific package
pnpm --filter @buoy-design/cli build
pnpm --filter @buoy-design/core build

# Run CLI locally (after building)
node apps/cli/dist/bin.js <command>

# Type checking
pnpm typecheck

# Run tests
pnpm test

# Format code
pnpm format

# Watch mode development
pnpm dev
```

## Architecture

This is a TypeScript monorepo using pnpm workspaces and Turbo.

### Package Structure

```
apps/cli/          # @buoy-design/cli - CLI application (entry point: bin.js)
packages/core/     # @buoy-design/core - Domain models, drift detection engine
packages/scanners/ # @buoy-design/scanners - Framework-specific code scanners (React, Vue, Svelte, Angular, Tailwind, etc.)
```

### Key Data Flow

1. **CLI commands** (`apps/cli/src/commands/`) parse args and orchestrate
2. **Scanners** (`packages/scanners/`) extract Components and DesignTokens from source files
3. **SemanticDiffEngine** (`packages/core/src/analysis/`) compares sources and produces DriftSignals
4. **Reporters** (`apps/cli/src/output/`) format output (table, JSON, markdown)
5. **Integrations** (`apps/cli/src/integrations/`) post results (GitHub PR comments)

### Core Domain Models (packages/core/src/models/)

- **Component**: Represents UI components from any framework (React, Vue, Svelte, etc.)
- **DesignToken**: Color, spacing, typography values from CSS/JSON/Figma
- **DriftSignal**: A detected issue (hardcoded-value, naming-inconsistency, deprecated-pattern, etc.)

### Built-in Scanners

All framework scanners are built-in (no plugins needed):
- **React/Vue/Svelte/Angular** - Component scanning in `packages/scanners/src/git/`
- **Tailwind** - Config parsing and arbitrary value detection in `packages/scanners/src/tailwind/`
- **Tokens** - CSS/SCSS/JSON token extraction in `packages/scanners/src/git/token-scanner.ts`
- **Templates** - Blade/ERB/Twig template scanning

### Optional Integrations

External services that require API keys are in `packages/scanners/`:
- **Figma** - Connect to Figma API for token comparison
- **Storybook** - Scan stories for component coverage

## CLI Commands

```
buoy
├── show                    # Read design system info (for AI agents)
│   ├── components          # Components in codebase
│   ├── tokens              # Design tokens found
│   ├── drift               # Design system violations
│   ├── health              # Health score
│   ├── history             # Scan history
│   └── all                 # Everything combined
├── drift                   # Drift detection and fixing
│   ├── scan                # Scan codebase for components/tokens
│   ├── check               # Pre-commit drift check
│   ├── fix                 # Suggest/apply fixes
│   └── ignore              # Ignore existing drift
│       ├── all             # Ignore all current drift (requires --reason)
│       ├── show            # View ignored drift signals
│       ├── add             # Add new drift to ignore list (requires --reason)
│       └── clear           # Remove ignore list
├── begin                   # Interactive wizard
├── dock                    # Dock tools into your project
│   ├── config              # Create .buoy.yaml
│   ├── skills              # Create AI agent skills
│   ├── agents              # Set up AI agents
│   ├── context             # Generate CLAUDE.md context
│   ├── hooks               # Set up git hooks
│   ├── commands            # Install Claude slash commands
│   ├── plugins             # Show available scanners
│   ├── tokens              # Generate/export design tokens
│   │   ├── compare         # Compare token sources
│   │   └── import          # Import tokens from Figma/CSS
│   └── graph               # Build design system knowledge graph
│       └── learn           # Learn patterns from codebase
└── ahoy                    # Cloud features
    ├── login               # Authenticate
    ├── logout              # Sign out
    ├── status              # Account + bot + sync status
    ├── github              # Set up GitHub PR bot
    ├── gitlab              # Set up GitLab PR bot (soon)
    ├── billing             # Manage subscription
    └── plans               # Compare pricing
```

### For AI Agents (primary interface)

| Command | Purpose |
|---------|---------|
| `buoy show components` | List components found in codebase |
| `buoy show tokens` | List design tokens |
| `buoy show drift` | List design system violations |
| `buoy show health` | Health score (0-100) |
| `buoy show all` | Everything in one call |
| `buoy show history` | Past scan results |

All `show` subcommands output JSON by default.

### Setup & CI

| Command | Purpose |
|---------|---------|
| `buoy begin` | Interactive wizard to get started |
| `buoy dock` | Configure project (config, agents, hooks) |
| `buoy dock config` | Create .buoy.yaml |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ahoybuoy/buoy](https://github.com/ahoybuoy/buoy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
