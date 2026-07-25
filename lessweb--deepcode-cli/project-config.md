---
trigger: always_on
description: This is an **npm workspaces monorepo**. Packages live under `packages/`.
---

# Repository Guidelines

## Project Structure & Module Organization

This is an **npm workspaces monorepo**. Packages live under `packages/`.

```
packages/
├── core/src/               # LLM session, tool execution, shared utilities
│   ├── common/             # File I/O, permissions, telemetry, OpenAI client, shell utils, etc.
│   ├── tools/              # 7 built-in handlers (bash, read, write, edit, web-search, ask-user-question, update-plan)
│   ├── mcp/                # MCP client & manager (JSON-RPC lifecycle)
│   ├── session.ts          # SessionManager — LLM loop, compaction, tool orchestration
│   ├── prompt.ts           # System prompt builder & tool definitions
│   └── settings.ts         # Settings resolution from ~/.deepcode/settings.json
├── cli/src/                # Terminal UI (Ink/React)
│   ├── cli.tsx             # Entry point — renders AppContainer
│   ├── cli-args.ts         # CLI argument parsing (yargs: -p, -r, -v, -h)
│   ├── common/             # Update checker
│   ├── utils/              # stdio helpers, version, package info
│   ├── generated/          # Build-time git commit info
│   ├── ui/views/           # Top-level screens (App, PromptInput, SessionList, PermissionPrompt, WelcomeScreen, UpdatePrompt, McpStatusList, etc.)
│   ├── ui/components/      # Reusable Ink components (MessageView, DropdownMenu, ModelsDropdown, etc.)
│   ├── ui/core/            # Prompt buffer, slash commands, file mentions, clipboard, undo/redo
│   ├── ui/hooks/           # Custom hooks (cursor, history navigation, paste handling, terminal input, statusline)
│   ├── ui/contexts/        # React contexts (AppContext, RawModeContext)
│   ├── ui/statusline/      # Pluggable statusline providers (command, module)
│   ├── ui/utils/            # Shared UI utilities (writing, formatting)
│   └── tests/              # UI-focused tests with run-tests.mjs runner
├── vscode-ide-companion/   # VSCode extension companion
│   └── src/                # extension.ts, provider.ts, utils.ts
docs/                       # User-facing documentation (configuration, MCP, notify, permissions)
scripts/                    # Build, release, and packaging scripts
dist/                       # Bundled CLI output — single-file dist/cli.js (gitignored)
dist/bundled/               # Bundled skills & references shipped with the CLI
```

Templates for tool descriptions and prompts are at `packages/cli/dist/templates/` (copied during build from `packages/core/templates/`). Built-in skills are under `packages/cli/dist/bundled/`.

## Build, Test, and Development Commands

All commands run from the repo root.

| Command | What it does |
|---|---|
| `npm run typecheck` | TypeScript type checking across all workspaces |
| `npm run lint` | ESLint across `packages/*/src/**/*.{ts,tsx}` + `scripts/*.js` |
| `npm run lint:fix` | ESLint with auto-fix |
| `npm run format` | Prettier on all source files |
| `npm run format:check` | Prettier in check-only mode |
| `npm run check` | Runs typecheck + lint + format:check together |
| `npm run build` | Orchestrates full build (scripts/build.js) — compiles core + bundles CLI + copies assets |
| `npm run bundle` | Generates git commit info + esbuild bundle + copies bundled assets |
| `npm run build:vscode` | Builds the VSCode extension companion |
| `npm test` | Runs all workspace tests (`npm run test --workspaces --if-present`) |
| `npm run start` | Runs the locally built CLI (`scripts/start.js`) |
| `npm run build-and-start` | Builds then starts the CLI |
| `npm run clean` | Removes generated files and dist directories |
| `npm run release:version` | Bumps version across all packages |
| `npm run prepare:package` | Prepares the CLI package for distribution |
| `npm run prepare:vscode` | Prepares the VSCode extension for distribution |

To run a **single test file** within a package:
```
node packages/core/src/tests/run-tests.mjs packages/core/src/tests/session.test.ts
node packages/cli/src/tests/run-tests.mjs packages/cli/src/tests/slash-commands.test.ts
```

Run the CLI locally for manual testing: `node packages/cli/dist/cli.js` (after `npm run bundle`).

## Coding Style & Naming Conventions

- **Indentation**: 2 spaces, no tabs
- **Quotes**: Double quotes (`"`)
- **Semicolons**: Required
- **Trailing commas**: `es5` (objects, arrays, etc.)
- **Line width**: 120 characters max
- **Line endings**: LF only

**TypeScript**: Strict mode enabled (`strict: true`). Use `import type` for type-only imports (`@typescript-eslint/consistent-type-imports`). Unused variables prefixed with `_` are allowed (`argsIgnorePattern: "^_"`). Target ES2022, module ESNext with bundler resolution. JSX is `react-jsx`.

**Formatting/Linting**: Prettier (double quotes, 2-space indent, semicolons) + ESLint (typescript-eslint, react-hooks). Run `npm run check` before pushing. On commit, Husky + lint-staged auto-formats staged `*.{ts,tsx,js,mjs,cjs,jsx}` and `*.json` files.

**File naming**: `kebab-case.ts` for modules, `kebab-case.tsx` for React/Ink components. Test files: `*.test.ts` (always kebab-case).

## Testing Guidelines

- **Framework**: Node.js native test runner (`node:test`) with `tsx` for TypeScript
- **Assertions**: `node:assert/strict`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lessweb/deepcode-cli](https://github.com/lessweb/deepcode-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
