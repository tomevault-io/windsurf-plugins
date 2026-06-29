---
trigger: always_on
description: This document provides guidance for AI coding assistants (Claude, GitHub Copilot, etc.) and developers working on linkinator. It documents architectural decisions, common pitfalls, and important considerations to prevent regressions.
---

# Agent Guide for Linkinator

This document provides guidance for AI coding assistants (Claude, GitHub Copilot, etc.) and developers working on linkinator. It documents architectural decisions, common pitfalls, and important considerations to prevent regressions.

## Table of Contents

1. [Build & Package Structure](#build--package-structure)
2. [Dual Runtime Targets](#dual-runtime-targets)
3. [Critical Files & Dependencies](#critical-files--dependencies)
4. [Common Pitfalls](#common-pitfalls)
5. [Testing Requirements](#testing-requirements)
6. [Release Process](#release-process)

---

## Build & Package Structure

### Directory Layout

```
linkinator/
├── src/              # TypeScript source files
│   ├── cli.ts       # CLI entry point
│   ├── index.ts     # Library entry point
│   └── ...
├── build/           # Compiled JavaScript output (gitignored)
│   ├── package.json # AUTO-GENERATED copy from root
│   ├── src/
│   │   ├── cli.js   # Compiled CLI
│   │   └── ...
│   └── test/
├── package.json     # Root package metadata
└── test/            # Test files
```

### TypeScript Compilation

The project uses TypeScript with these key settings (`tsconfig.json`):

```json
{
  "compilerOptions": {
    "rootDir": ".",
    "outDir": "build",
    "resolveJsonModule": true  // ⚠️ IMPORTANT: Copies JSON imports to build/
  }
}
```

**Important**: When TypeScript sees `import foo from './bar.json'`, it:
1. Copies `bar.json` to the build directory (maintaining relative structure)
2. Generates a `.d.ts` declaration file for type safety

### NPM Package Contents

The `files` array in `package.json` controls what gets published to npm:

```json
{
  "files": [
    "build/src",           // All compiled source
    "build/package.json"   // REQUIRED for CLI version info
  ]
}
```

**⚠️ CRITICAL**: The npm package does NOT include:
- Root `package.json`
- `build/test/` directory
- Any files outside `build/src/` and `build/package.json`

---

## Dual Runtime Targets

Linkinator supports **two distinct runtime environments**:

### 1. NPM Package (Node.js)

```bash
npm install linkinator
linkinator ./docs
```

**How it works**:
- User installs via npm
- Gets contents specified in `files` array
- Runs `node_modules/linkinator/build/src/cli.js`
- Imports resolve to files in `node_modules/linkinator/`

### 2. Compiled Binaries (Standalone Executables)

```bash
./linkinator-linux ./docs
```

**How it works**:
- Created via `bun build --compile`
- Bundles all JavaScript and dependencies into a single executable
- No access to filesystem for imports (everything is embedded)
- Uses embedded JSON imports from build time

---

## Critical Files & Dependencies

### CLI Version Information

**File**: `src/cli.ts:6`

```typescript
import packageJson from '../package.json' with { type: 'json' };
```

**Why this exists**:
- Originally added in #760 to fix `--version` flag in compiled binaries
- Before this, meow tried to read package.json at runtime, which failed in binaries

**How it works**:
1. **During build**: TypeScript copies `package.json` → `build/package.json`
2. **Compiled to**: `import packageJson from '../package.json'` (relative path unchanged)
3. **At runtime** (npm): `build/src/cli.js` imports `build/package.json` ✅
4. **At compile time** (binaries): Bundler embeds the JSON content ✅

**⚠️ CRITICAL REQUIREMENT**:
- `build/package.json` MUST be included in the npm package
- Without it, npm users get: `Cannot find module '/node_modules/linkinator/build/package.json'`
- This caused issue #763

### Package File Checklist

When modifying packaging, verify:
- [ ] `"build/package.json"` is in the `files` array
- [ ] The import path in `cli.ts` is `../package.json` (not `../../`)
- [ ] `tsconfig.json` has `resolveJsonModule: true`
- [ ] `npm pack --dry-run` includes `build/package.json`

---

## Common Pitfalls

### 🚨 Pitfall 1: Removing `build/package.json` from `files` Array

**What happens**: NPM users can't run the CLI
**Error**: `Cannot find module '/node_modules/linkinator/build/package.json'`
**Why**: The CLI imports from `../package.json` relative to `build/src/cli.js`
**How to avoid**: Never remove `build/package.json` from the `files` array without changing how the CLI gets version info

### 🚨 Pitfall 2: Changing the Import Path to `../../package.json`

**What happens**: Works for npm, breaks for compiled binaries
**Why**:
- From `src/cli.ts`, `../../package.json` would look outside the project
- TypeScript won't copy a file from outside the project
- Compiled binaries won't have the package.json embedded
**How to avoid**: Always import from `../package.json` (one level up from src/)

### 🚨 Pitfall 3: Version Drift Between Root and Build

**What happens**: `--version` shows wrong version
**Why**:
- Someone edited `package.json` without rebuilding
- `build/package.json` has stale version
**How to avoid**:
- Always run `npm run build` after version changes
- The release workflow rebuilds automatically
- Test `test.cli.ts` verifies version consistency

### 🚨 Pitfall 4: Adding JSON Imports Without Updating `files` Array


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JustinBeckwith/linkinator](https://github.com/JustinBeckwith/linkinator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
