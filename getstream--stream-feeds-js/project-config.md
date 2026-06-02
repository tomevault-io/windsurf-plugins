---
trigger: always_on
description: Guidance for AI coding agents (Copilot, Cursor, Aider, Claude, etc.) working in this repository. Human readers are welcome, but this file is written for tools.
---

Guidance for AI coding agents (Copilot, Cursor, Aider, Claude, etc.) working in this repository. Human readers are welcome, but this file is written for tools.

### Repository purpose

This repo hosts Stream's Feeds JS SDKs in TypeScript. It provides:
• A comprehensive JS, React and React Native client for Stream's Feeds (v3) API
• Support for real-time feeds, notifications, and moderation

Agents should prioritize backwards compatibility, API stability, and high test coverage when changing code.

### Tech & toolchain

• Language: TypeScript, React, React Native
• Package managers: Yarn with workspaces
• Node.js: 22.0.0 or newer
• Runtime targets: React 18+, React Native 0.73+
• CI: GitHub Actions (assume PR validation on build + tests + lint)
• Linters & docs: ESLint + Prettier
• Build tools: Vite, TypeScript (feeds-client); react-native-builder-bob for React Native SDK

### Project layout (monorepo)

```text
packages/
  feeds-client/                # Core Feeds API client
    src/
      activity-with-state-updates/  # Activity state management
      bindings/                # Framework bindings
      common/                  # Common utilities (API client, state management, real-time)
      feed/                    # Feed management and event handlers
      feeds-client/            # Main FeedsClient class
      gen/                     # Generated API clients and models
      utils/                   # Utility functions
  react-sdk/                   # React SDK wrapper with hooks and contexts
  react-native-sdk/            # React Native SDK wrapper
sample-apps/
  react-demo/                  # Next.js demo application (stream-feeds-react-demo)
  react-native/                # React Native sample application (ExpoTikTokApp)
```

Use the closest folder's patterns and conventions when editing.

### Local setup (Monorepo)

1.  Ensure Node.js 22+ is installed
2.  Install dependencies: `yarn install`
3.  Build all packages: `yarn build:all`
4.  Run tests: `yarn test:ci:all`

### Package-specific setup

For individual packages:

```bash
# Core feeds client
yarn workspace @stream-io/feeds-client run build
yarn workspace @stream-io/feeds-client run test

# React SDK
yarn workspace @stream-io/feeds-react-sdk run build

# React Native SDK
yarn workspace @stream-io/feeds-react-native-sdk run build
```

### Available Scripts

Root-level scripts:
• `build:all` - Build all packages
• `build:libs` - Build only library packages (no sample apps)
• `test:ci:all` - Run all tests with coverage
• `test:ci:libs` - Run library tests only
• `lint:all` - Lint all packages
• `lint:all:fix` - Fix linting issues
• `generate` - Regenerate API clients from OpenAPI specs

Package-specific scripts (check individual package.json files):
• `build` - Build the package
• `test` - Run tests with Vitest
• `test:unit` - Run unit tests only
• `test-ci` - Run tests with coverage
• `clean` - Clean build artifacts

Agents should check package.json scripts before running commands.

### Build & test commands (CLI)

Use yarn/npm for day-to-day work; use CLI for CI parity & agent automation.

Build all packages:

```bash
yarn build:all
```

Build only libraries (no sample apps):

```bash
yarn build:libs
```

Run all tests:

```bash
yarn test:ci:all
```

Run library tests only:

```bash
yarn test:ci:libs
```

Run linting:

```bash
yarn lint:all
```

Fix linting issues:

```bash
yarn lint:all:fix
```

Generate API clients:

```bash
yarn generate
```

Linting & formatting
• ESLint and Prettier are configured for this project. Run before committing:

```bash
yarn prettier --write <changed files>   # Format changed files with prettier
yarn lint:all
yarn lint:all:fix
```

• **Always run `yarn prettier --write` on changed files before considering changes complete.**
• Respect eslint.config.mjs and .prettierrc configurations. Do not suppress rules broadly; justify and scope exceptions.
• The project uses lint-staged with Husky for pre-commit hooks.
• Each package may have its own linting configuration - check individual package.json files.

Commit / PR conventions
• Use conventional commit format for commit messages.
• Keep PRs small and focused; include tests.
• Follow the project’s “zero warnings” policy—fix new warnings and avoid introducing any.
• Ensure public API changes include docs and migration notes.

Testing policy
• Add/extend tests in each package's test directories with .test.ts suffix.
• Cover:
• Core FeedsClient and Feed classes
• Event handlers and state management - see ai-docs/ai-state-management for details
• React hooks and contexts (react-sdk, react-native-sdk)
• Utility functions (token creation, rate limiting, search)
• Generated API clients and their interactions
• Integration tests live in `packages/feeds-client/__integration-tests__/`

**Integration tests: HTTP + WebSocket**  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GetStream/stream-feeds-js](https://github.com/GetStream/stream-feeds-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
