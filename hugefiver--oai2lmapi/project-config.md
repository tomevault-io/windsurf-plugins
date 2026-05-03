---
trigger: always_on
description: **OAI2LMApi** bridges OpenAI-compatible APIs to VSCode's Language Model API for GitHub Copilot Chat and other AI-powered features.
---

# Copilot Instructions for OAI2LMApi

## Project Overview

**OAI2LMApi** bridges OpenAI-compatible APIs to VSCode's Language Model API for GitHub Copilot Chat and other AI-powered features.

### Technology Stack

- **Language**: TypeScript 5.9.3
- **Target Runtime**: Node.js 18+ (VSCode engine 1.107.0+)
- **Package Manager**: pnpm 10.x (REQUIRED - do not use npm or yarn)
- **Build Tool**: esbuild (via custom esbuild.js script)
- **Testing**: Mocha with @vscode/test-electron
- **Linting**: ESLint 9.x with TypeScript plugin

## Repository Structure

```
/
├── .github/workflows/          # CI/CD pipelines
│   ├── lint-test.yml           # Linting and testing workflow
│   ├── build-test-package.yml  # Build and package workflow
│   ├── prerelease-main.yml     # Prerelease on main push workflow
│   └── release.yml             # Release to marketplace workflow
├── .vscode/                    # VSCode workspace settings
│   ├── launch.json             # Debug configurations
│   ├── tasks.json              # Build tasks
│   └── extensions.json         # Recommended extensions
├── src/                        # Extension source code
├── out/                        # Build output (generated, gitignored)
├── assets/                     # Extension assets
├── scripts/                    # Build scripts
├── packages/                   # Workspace packages
│   └── model-metadata/         # Shared model metadata package
│       ├── src/index.ts        # Shared model metadata registry
│       └── package.json        # Shared metadata manifest and scripts
├── esbuild.js                  # Custom esbuild bundler
├── package.json                # Extension manifest and scripts
├── tsconfig.json               # TypeScript compiler settings
├── eslint.config.mjs           # ESLint config
├── pnpm-workspace.yaml         # Workspace configuration
└── README.md                   # Project overview
```

### Key Files

- **entry point**: `src/extension.ts` - exports `activate()` and `deactivate()`
- **extension manifest**: `package.json` - VSCode extension metadata, commands, and configuration
- **shared metadata**: `packages/model-metadata/src/index.ts` - model metadata registry used by all packages
- **esbuild.js**: bundles extension into `out/extension.js`
- **tsconfig.json**: TypeScript compiler settings for the extension

## Build & Development Workflow

### Prerequisites

**CRITICAL**: This project REQUIRES pnpm. Install it globally before starting:

```bash
npm install -g pnpm@10
```

### Initial Setup

1. **Install dependencies** (ALWAYS use frozen lockfile in CI/scripts):

   ```bash
   pnpm install --frozen-lockfile
   ```

   - Time: ~5-10 seconds (with cache)
   - Creates `node_modules/` with 545 packages
   - You may see warnings about ignored build scripts (safe to ignore)

> **Build note**: All VSCode extension commands (`check-types`, `lint`, `compile`, `test`, `package`) are run from the project root. Use `pnpm --filter @oai2lmapi/model-metadata` for the metadata package.

### Build Commands

#### Type Checking

```bash
pnpm run check-types
```

- Runs TypeScript compiler with `--noEmit` flag (no output, just validation)
- Time: ~2-3 seconds
- ALWAYS run this before committing code changes

#### Linting

```bash
pnpm run lint
```

- Runs ESLint on `src/**/*.ts`
- Time: ~1-2 seconds
- Must pass with zero errors before committing
- Key rules enforced:
  - TypeScript naming conventions (camelCase for variables, PascalCase for types)
  - Semicolons required
  - Curly braces for control statements
  - Strict equality (===)

#### Compile Extension

```bash
pnpm run compile
```

- Runs `check-types` then bundles with esbuild
- Output: `out/extension.js` (with source map)
- Time: ~2-3 seconds
- This is the primary build command for development
- **NOTE**: The compile step runs type checking first automatically

#### Compile Tests

```bash
pnpm run compile:tests
```

- Compiles TypeScript to JavaScript using tsc (not esbuild)
- Output: `out/**/*.js` including test files
- Time: ~2-3 seconds
- Required before running tests

#### Production Build

```bash
pnpm run vscode:prepublish
```

- Runs `check-types` + esbuild with `--production` flag
- Minified output, no source maps
- Removes console.log and debugger statements
- Automatically run by `pnpm run package`

### Testing

**IMPORTANT**: Tests require a display server (xvfb) because they launch VSCode.

#### Run All Tests

```bash
xvfb-run -a pnpm test
```

- On Linux CI: MUST use `xvfb-run -a` prefix
- On macOS/Windows: Can run `pnpm test` directly
- Runs pretest (compile:tests + lint) automatically
- Downloads VSCode 1.107.0 on first run (cached afterward)
- Time: ~20-30 seconds first run, ~10-15 seconds cached

#### Pretest

```bash
pnpm run pretest
```

- Runs `compile:tests` then `lint`
- Automatically executed before `pnpm test`

### Packaging

```bash
pnpm run package
```

- Creates `.vsix` file for distribution: `oai2lmapi-{version}.vsix`
- Runs `vscode:prepublish` automatically
- Uses `vsce package --no-dependencies` (dependencies bundled by esbuild)
- Output: ~52KB VSIX file
- Time: ~3-4 seconds

### Development Commands

#### Watch Mode

```bash
pnpm run watch
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hugefiver/OAI2LMApi](https://github.com/hugefiver/OAI2LMApi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
