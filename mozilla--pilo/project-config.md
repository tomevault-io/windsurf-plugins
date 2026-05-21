---
trigger: always_on
description: Pilo is an AI-powered web automation library and CLI tool that lets you control browsers using natural language. It is structured as a pnpm monorepo under `packages/`, with the root package (`@tabstack/pilo`) serving as both the workspace orchestrator and the publishable npm package.
---

# Pilo Project - Claude Instructions

## Project Overview

Pilo is an AI-powered web automation library and CLI tool that lets you control browsers using natural language. It is structured as a pnpm monorepo under `packages/`, with the root package (`@tabstack/pilo`) serving as both the workspace orchestrator and the publishable npm package.

## Monorepo Structure

```
pilo/
├── packages/
│   ├── core/        # pilo-core: automation engine
│   ├── cli/         # pilo-cli: CLI commands and config
│   ├── server/      # pilo-server: Hono-based server
│   └── extension/   # pilo-extension: WXT/React browser extension
├── scripts/         # Build, release, and CI scripts
├── dist/            # Assembled output (root build only)
├── package.json     # @tabstack/pilo workspace root + published package
└── pnpm-workspace.yaml
```

### Packages

| Path                 | npm name         | Description                                                                                 |
| -------------------- | ---------------- | ------------------------------------------------------------------------------------------- |
| `packages/core`      | `pilo-core`      | Core automation library. Browser-safe subset via `core.ts`; full Node.js API via `index.ts` |
| `packages/cli`       | `pilo-cli`       | CLI entry point, commands, and config integration                                           |
| `packages/server`    | `pilo-server`    | Hono-based server component                                                                 |
| `packages/extension` | `pilo-extension` | WXT-based browser extension with React UI                                                   |
| root                 | `@tabstack/pilo` | Published npm package: bundles core + CLI + pre-built extension                             |

### Core Package Layout (`packages/core/src/`)

- `browser/` - Browser automation implementations
- `config/` - Config system (see Config System section)
- `tools/` - AI agent tools
- `search/` - Search utilities
- `utils/` - Utility functions
- `index.ts` - Full Node.js public API (barrel)
- `core.ts` - Browser-safe public API (barrel, no Node.js deps)

### Extension Package Layout (`packages/extension/src/`)

- `ui/` - Sidepanel React components, hooks, stores
- `background/` - Service worker (AgentManager, ExtensionBrowser)
- `content/` - Content script entry (imports from `shared/`)
- `shared/` - Types, utils, and stores used across multiple entrypoints
- `entrypoints/` (root of extension package) - WXT entrypoints

## Common Commands

### Full Validation

```bash
pnpm run check          # typecheck (pretest + format:check + per-package typechecks) + test across all packages
```

### Testing

```bash
pnpm -r run test                                       # Run all tests across all packages
pnpm --filter pilo-core run test                      # Test core only
pnpm --filter pilo-cli run test                       # Test CLI only
pnpm --filter pilo-extension run test                 # Test extension only (unit, vitest)
pnpm --filter pilo-server run test                    # Test server only
pnpm --filter pilo-extension run test:e2e             # Extension e2e tests (headed, Playwright)
pnpm --filter pilo-extension run test:e2e:headless    # Extension e2e tests (headless)
```

### Building

```bash
pnpm run build          # Full assembly: prebuild (core + extension) then tsup compiles core + CLI into root dist/
pnpm -r run build       # Build all packages individually
pnpm run clean          # Clean all packages and root dist/
```

### Development

```bash
pnpm run dev:server                           # Run dev server (tsx watch)
pnpm run dev:extension -- --chrome            # Extension dev (Chrome, WXT HMR)
pnpm run dev:extension -- --firefox           # Extension dev (Firefox, WXT HMR)
pnpm run dev:extension -- --chrome --tmp      # Extension dev with temporary profile
pnpm run format                               # Format all code with Prettier
pnpm run format:check                         # Check formatting
pnpm run typecheck                            # Generate ariaTree bundle + format:check + typecheck all packages
```

### Installation & Setup

```bash
pnpm install              # Install all workspace dependencies
pnpm playwright install   # Install browser automation drivers
```

### Running Pilo CLI Locally

```bash
pnpm pilo run "task"            # Run an automation task
pnpm pilo config init           # Initialize config interactively
pnpm pilo config set <key> <value>
pnpm pilo config get <key>
pnpm pilo config list
pnpm pilo config show
pnpm pilo config unset <key>
pnpm pilo config reset
pnpm pilo extension install chrome [--tmp]
pnpm pilo extension install firefox [--tmp] [--firefox-binary <path>]
```

## Development Workflow

1. **Before making changes**: Run `pnpm run check` to verify a clean baseline.
2. **After making changes**:
   - Run `pnpm run format` to format code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mozilla/pilo](https://github.com/mozilla/pilo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
