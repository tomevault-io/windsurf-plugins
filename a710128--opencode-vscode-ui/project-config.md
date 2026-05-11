---
trigger: always_on
description: This repo is a standalone VS Code extension for browsing and operating OpenCode sessions per workspace folder. It starts `opencode serve` for each workspace, shows grouped sessions in the Activity Bar sidebar, and opens session tabs with a local webview UI. The local `opencode/` directory is only a symlink to the upstream repo for reference; do not import from it, depend on it at runtime, or commit it.
---

# OpenCode UI

This repo is a standalone VS Code extension for browsing and operating OpenCode sessions per workspace folder. It starts `opencode serve` for each workspace, shows grouped sessions in the Activity Bar sidebar, and opens session tabs with a local webview UI. The local `opencode/` directory is only a symlink to the upstream repo for reference; do not import from it, depend on it at runtime, or commit it.

- `src/`: extension source code
  - `src/core/`: workspace runtime, SDK access, commands, event streaming, session state, and tab orchestration
  - `src/sidebar/`: TreeDataProvider, tree items, focused session tracking, and sidebar webviews
  - `src/panel/`: session panel provider, serializer, host-side controller logic, and webview HTML shell
  - `src/bridge/`: typed host/webview message contracts and panel constants
- `images/`: extension icons and activity bar assets
- `dist/`: compiled extension and webview bundle output
- `PLAN.md`: implementation plan and milestone tracking
- `AGENTS.md`: instructions for agentic coding agents in this repository
- `.memory/`: local memory log, git ignored
- `opencode/`: local upstream symlink for reference only, git ignored

## MOST IMPORTANT RULES

- DO NOT modify AGENTS.md unless the user permits it by explicitly asking you to do so.
- When you adding new feature or fixing a bug, please consider about how to minimize the impact on the existing code, which means modify codes as less as possible.
- When you want to ask user for making decisions, if there is a `question` tool or `ask_user` tool, prefer to use it instead of asking the user directly. If there is no suitable tool, ask the user directly.
- Before making any changes, write a TODO list to remind yourself to implement the features later, if there is a `todo` tool, prefer to use it instead of writing the TODO list manually.
- Before you start writing code, you need to inform the user of the solution you intend to adopt. Only after discussing and confirming with the user should you begin the work. And also remember that DO NOT ASK ANY QUESTIONS IN SUBAGENTS.

## Run / Build / Lint / Test

- Package manager: `bun` (`packageManager: bun@1.3.10`).
- Install dependencies: `bun install`.
- Main development validation: `bun run check-types && bun run lint && bun run compile`.
- Type-check only: `bun run check-types`.
- Lint only: `bun run lint`.
- Development build: `bun run compile`.
- Production package build: `bun run package`.
- Prepublish hook: `bun run vscode:prepublish`.
- Watch esbuild bundle: `bun run watch:esbuild`.
- Watch TypeScript only: `bun run watch:tsc`.
- Current test command: `bun run test`.
- Default test scope: `bun run test` runs `bun test ./src/panel ./src/test` so only this extension's local tests run, not the upstream `opencode/` symlink tree.
- Current single-test support: `bun test path/to/file.test.ts`.
- Test placement convention:
  - Keep module-local unit tests next to the code they cover, such as `src/panel/**/foo.test.ts`.
  - Put cross-module, integration-style, and parity tests under `src/test/`.
  - Do not add new tests under the local `opencode/` symlink.
- Build pipeline details from `package.json`:
  - `compile`: runs type-check, lint, then `node esbuild.js`
  - `package`: runs type-check, lint, then `node esbuild.js --production`
- Prefer validating code changes before handing work back, especially for edits under `src/core/`, `src/panel/`, `src/sidebar/`, or `src/bridge/`.
- No Cursor rules were found in `.cursor/rules/` or `.cursorrules` as of this file update.
- No Copilot rules were found in `.github/copilot-instructions.md` as of this file update.
- Do not run repo workflow commands against the local `opencode/` symlink.

## Code Style

- Language and toolchain:
  - The repo is TypeScript-first with React TSX for webviews and CSS for styling.
  - `tsconfig.json` uses `strict: true`, `module: Node16`, `moduleResolution: Node16`, `target: ES2022`, and `jsx: react-jsx`.
  - Preserve strict typing; do not weaken types with `any`, broad casts, or untyped message payloads.
- Imports:
  - Keep imports simple: external modules first, then local relative imports.
  - Use `import * as vscode from "vscode"` for VS Code APIs.
  - Use `import type` for type-only imports when possible.
  - Prefer short relative paths that match the existing folder structure; do not introduce alias systems casually.
- Formatting:
  - Match the existing style: double quotes, semicolons omitted, and 2-space indentation.
  - Prefer small functions, early returns, and lightweight helpers over deep nesting.
  - Keep lines readable; extract helpers instead of piling on conditional branches.
  - Add comments only when a block is not obvious from surrounding code.
- Naming:
  - Classes, React components, and exported types use PascalCase.
  - Functions, methods, variables, fields, and hooks use camelCase.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [a710128/opencode-vscode-ui](https://github.com/a710128/opencode-vscode-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
