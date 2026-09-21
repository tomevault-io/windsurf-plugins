---
trigger: always_on
description: **agent2linear** is a TypeScript-based command-line tool for creating and managing Linear projects, issues, labels, workflow states, and other entities via the Linear GraphQL API. Designed for AI agents and automation workflows.
---

# agent2linear - Linear CLI Tool

  ## Project Overview
  **agent2linear** is a TypeScript-based command-line tool for creating and managing Linear projects, issues, labels, workflow states, and other entities via the Linear GraphQL API. Designed for AI agents and automation workflows.

  **CLI Commands:**
  - `agent2linear` - Full command name
  - `a2l` - Short alias for convenience

  **Technology Stack:**
  - TypeScript (ES2022, ESNext modules)
  - Commander.js (CLI framework)
  - Ink (React-based terminal UI for interactive modes)
  - Linear SDK (@linear/sdk)
  - Build: tsup (TypeScript bundler)
  - Task runner: turbo

  ## Prerequisites & Setup

  ### Required Environment Variables
  ```bash
  export LINEAR_API_KEY=lin_api_xxxxxxxxxxxx

  Installation & Build

  # For development
  npm install
  npm run build

  # For end users (npm package)
  npm install -g agent2linear

  Running the CLI

  # After build (both commands work identically)
  agent2linear --help
  a2l --help

  # Or via node directly during development
  node dist/index.js --help

  Development Workflow

  Build Commands

  npm run build        # Production build (tsup)
  npm run dev          # Watch mode for development
  npm run lint         # ESLint check
  npm run typecheck    # TypeScript type checking
  npm run format       # Prettier formatting

  Architecture

  - Entry point: src/index.ts → compiles to dist/index.js (with shebang)
  - CLI definition: src/cli.ts - all command registration
  - Commands: src/commands/<entity>/<action>.ts(x) (.tsx for Ink components)
  - Libraries: src/lib/ - shared utilities (aliases, config, linear-client, etc.)
  - Types: src/lib/types.ts - TypeScript interfaces for all entities

  ## Code Intelligence Tooling

  This repo enables two Claude Code plugins for code intelligence. They are
  declared in the committed `.claude/settings.json` (`enabledPlugins`), so they
  apply to everyone who clones the repo:

  ```json
  {
    "enabledPlugins": {
      "typescript-lsp@claude-plugins-official": true,
      "ast-grep@ast-grep-marketplace": true
    }
  }
  ```

  **Note:** `enabledPlugins` is read at session startup; after editing
  `.claude/settings.json`, start a fresh session for changes to take effect.
  Personal/machine-specific overrides belong in `.claude/settings.local.json`
  (gitignored, not shared).

  ### TypeScript LSP (semantic navigation)

  Use for questions that require **type resolution** — "where is X defined / used",
  type/JSDoc info, refactor-grade reference analysis, and call hierarchy. Resolves
  this repo's ESM `.js`-extension imports back to their `.ts` sources via
  `tsconfig.json`.
  - Operations: `goToDefinition`, `findReferences`, `hover`, `documentSymbol`,
    `workspaceSymbol`, `goToImplementation`, `incomingCalls`/`outgoingCalls`.
  - Plugin: `typescript-lsp@claude-plugins-official`.

  ### ast-grep (structural pattern matching)

  Use for **structural** queries that ignore types — "every exported async function
  that awaits", "all `console.error(...)` call sites". Complements the LSP; it
  matches on AST shape, not semantics.
  - Requires the `ast-grep` binary on PATH (`brew install ast-grep`).
  - Match the grammar to the file's role: `--lang ts` for pure-logic `.ts` files,
    `--lang tsx` for Ink/React `.tsx` command files (JSX is a different grammar).
  - For relational YAML rules (`has`/`inside`), add `stopBy: end` so the traversal
    reaches descendants buried in function bodies.
  - Plugin: `ast-grep@ast-grep-marketplace` — docs:
    https://github.com/ast-grep/claude-skill

  ## Icon Handling (v0.13.2+)

  **IMPORTANT**: Icons are NOT validated client-side.

  - Icons are passed directly to Linear API for server-side validation
  - The curated icon list (src/lib/icons.ts) is for discovery only, not validation
  - Investigation confirmed Linear's API has no endpoint for the standard icon catalog
  - The `emojis` GraphQL query only returns custom organization emojis (user-uploaded)
  - See README.md "Icon Usage" section for user documentation
  - See MILESTONES.md M14.6 for complete investigation and rationale
  - See src/commands/project/create.tsx:208 for inline code documentation

  When implementing new commands with icon support:
  - Do NOT add client-side icon validation
  - Pass icon values directly to Linear API
  - Let Linear return errors for invalid icons
  - Reference the Icon Handling section for future developers

  Testing

  Test Suite Location

  All tests are in tests/scripts/ directory.

  Test Philosophy

  Integration tests using real Linear API, NOT unit tests.

  Tests create actual Linear entities with TEST_<timestamp>_ prefix and generate cleanup scripts (since delete commands aren't fully implemented yet).

  Running Tests

  Prerequisites

  1. LINEAR_API_KEY environment variable must be set
  2. Project must be built: npm run build
  3. Linear workspace should have at least one team

  Run All Tests

  cd tests/scripts
  ./run-all-tests.sh                # Run all project + issue tests
  ./run-all-tests.sh --project-only # Run only project tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smorinlabs/agent2linear](https://github.com/smorinlabs/agent2linear) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
