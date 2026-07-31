---
trigger: always_on
description: This file contains instructions for AI agents working with the `@diplodoc/cli` project.
---

# AGENTS.md

This file contains instructions for AI agents working with the `@diplodoc/cli` project.

## Module Documentation

If a module contains a `MODULE.md` file, it is considered part of the module's context and should be consulted when working with that module. These files provide module-specific documentation, architecture decisions, and implementation details that complement the general project documentation.

## Project Description

`@diplodoc/cli` is a CLI tool for building documentation from Markdown files with Yandex Flavored Markdown (YFM) support. The project allows you to build full-fledged documentation with navigation, internal transitions, and full YFM support.

## Project Structure

### Main Directories

- `src/` — source code of the project
  - `commands/` — CLI commands (build, publish, translate)
  - `core/` — core logic (config, logger, markdown, toc, etc.)
  - `extensions/` — extensions (openapi, generic-includer, local-search)
  - `steps/` — **DEPRECATED** processing steps (do not add new code here)
- `build/` — compiled code (generated during build)
- `lib/` — libraries (generated during build)
- `tests/` — project tests
  - `e2e/` — end-to-end tests
  - `cases/` — test cases
  - `fixtures/` — test fixtures
  - `mocks/` — test mocks

### CLI Commands

1. **build** — main command for building documentation
   - Location: `src/commands/build/`
   - Subcommands and features are located in `src/commands/build/features/`

2. **publish** — publishing documentation
   - Location: `src/commands/publish/`

3. **translate** — translating documentation
   - Location: `src/commands/translate/`

4. **content** — render a single file to stdout/file (md or html content fragment)
   - Location: `src/commands/content/`
   - Reuses the build `Run`, its `normalize`/`validate` and `buildConfigDefaults` (`src/commands/build/config.ts`), `MarkdownCollector`/`SELF_CONTAINED` (md) and `getBaseMdItPlugins` (html).
   - Root/file resolution is the pure `resolveContentConfig` (`src/commands/content/config-resolve.ts`): picks the project root from `-c`'s directory or CWD (falling back to the file's own dir when it lives outside that root), sets `config.input`/`output` to that root with `originAsInput = true`, stores the target as `config.file`, and populates `config.template` itself (build's `Templating` feature is not run).
   - stdout has two modes: default wraps the result in `CONTENT_START`/`CONTENT_END` markers (so it can be fished out of the framework banners); `--raw` (`config.raw`) emits only the content. In raw mode `src/index.ts` also skips the version/timer/completion banners via `isRawContentRun(argv)` (exported from the content command). `-o` always writes raw (no markers), regardless of `--raw`.
   - Note: `resolveConfig` treats `ENOTDIR` like `ENOENT` so a single-file `--input` doesn't break config discovery.
   - Tests: unit specs co-located in `src/commands/content/*.spec.ts` (resolution, render md/html, emit stdout/`-o`, watch paths/rebuild, `ContentWatcher`) drive Sonar coverage; e2e lives in `tests/e2e/content.spec.ts` (out-of-process, not counted by Sonar).

## Tech Stack

- **Language**: TypeScript
- **Runtime**: Node.js >= 22
- **Testing**: Vitest
- **Build**: esbuild
- **Linting**: ESLint (via `@diplodoc/lint`)

## Development Commands

```bash
# Build project
npm run build

# Run tests
npm test

# Run tests in watch mode
npm run test:watch

# Run E2E tests
npm run e2e

# Type checking
npm run typecheck

# Linting
npm run lint
npm run lint:fix
```

## Working with Code

### Imports

The project uses path aliases:

- `~/` — points to `src/`
- Example: `import {Program} from '~/core/program'`

### Architecture

1. **Program** — main class that manages command execution
   - Located in `src/commands/index.ts`
   - Uses decorator pattern (`@withConfigDefaults`)

2. **Commands** — CLI commands inherit from `BaseProgram`
   - Each command can have its own options, hooks, and handlers

3. **Features** — functionality is grouped into features
   - Examples: `output-html`, `search`, `watch`, etc.
   - Each feature can have its own configuration and hooks

4. **Hooks** — hook system for extending functionality
   - Uses `tapable` library

## Command Structure

Every command should follow a well-defined structure to ensure maintainability and separation of concerns.

### Command Organization

- **Main logic** is concisely gathered in `index.ts`
- **Implementation** is divided into separate functionalities through `features/`
- **Features** do not depend on each other
- A command can extract part of functionality into **services**, located in `services/` directory
- **Services** do not depend on each other
- If multiple features need to use the same computed data, such data (and logic for working with them) should be extracted into a service
- If multiple features use the same logic for processing their own data, such logic should likely be extracted into a common utility in the `utils/` directory at the same level as `features/` and `services/`
- **Utils** may depend on each other
- **Utils** cannot depend on anything outside of `utils/`

### Command Files

- **`index.ts`** — main entry point with concise core logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diplodoc-platform/cli](https://github.com/diplodoc-platform/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
