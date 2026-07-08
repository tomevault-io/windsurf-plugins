---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Architecture Overview

**Multi Module Flutter Tools** is a VS Code extension for running Flutter/Dart commands across multi-module workspaces. The architecture is:

### Core Layers

1. **Extension Entry** (`src/extension.ts`)
   - Registers all 15 commands in the command palette
   - Manages the output channel (streaming shell command output)
   - Orchestrates two main operation types: `runProjectOperation()` and `runWorkspaceOperation()`
   - Each operation tracks `OperationStats` (success/failure counts, duration, errors per module)

2. **UI Layer** (Webview Sidebar)
   - `MultiModuleViewProvider` + `MultiRepoViewProvider`: render sidebar sections (Cache, Packages, Git, Analysis, Run)
   - `BaseWebviewProvider` (base class): shared webview utilities, nonce generation, codicon loading
   - Commands are button clicks in the HTML sidebar → message posted to extension → command executed

3. **Shell Execution** (`runShellCommand()` in extension.ts)
   - Uses `child_process.spawn()` for streaming output (not buffered exec)
   - Respects VS Code `CancellationToken` — killing the process immediately on cancel
   - Applies FVM proxy if configured (`multiModuleFlutter.useFvm`)
   - Returns `{ ok: boolean; message?: string }`

4. **Project Discovery** (`src/repoDiscovery.ts`)
   - Scans workspace for `pubspec.yaml` files (Flutter modules)
   - Respects `multiModuleFlutter.maxDepth` and `multiModuleFlutter.excludeFolders` settings
   - Returns flat list of `ProjectInfo` (name, path)

5. **Notifications & Messages** (`src/notificationManager.ts`, `src/constants.ts`)
   - `NotificationManager`: shows VSCode popups (success/error/warning/info)
   - `constants.ts`: 26 error message categories + success templates (single source of truth)
   - Auto-categorizes completion notifications based on success/failure counts

6. **Type System** (`src/types.ts`)
   - `ProjectInfo`: `{ name, path }`
   - `OperationStats`: `{ successCount, failureCount, totalCount, durationMs, cancelled, errors }`
   - `CommandResult`, `NotificationConfig`: standardized across operations

### Command Flow Example
User clicks "Pub Get" → sidebar button posts message → extension receives `runCacheRepair` command → `runWorkspaceOperation()` called → for each workspace root, spawn `flutter pub get` → collect stats → notify completion with emoji (✅/⚠️/❌)

---

## Development Commands

### Build & Compilation
```bash
npm run compile        # TypeScript → dist/extension.js (includes esbuild minification)
npm run watch         # Continuous compile (watch mode)
npm run check-types   # TypeScript type checking (no emit)
npm run lint          # ESLint check
```

### Testing
```bash
npm test              # Run all 56 tests (Mocha via VSCode Test CLI)
npm run compile-tests # Compile tests only (tsc -p . --outDir out)
npm run watch-tests   # Watch test compilation
```

### Package & Release
```bash
npm run package       # Production build (check-types + lint + minify)
npm run vscode:prepublish  # Pre-publication hook (runs package)
```

### Single Test (if needed)
Edit the test file filter in `src/test/extension.test.ts` (e.g., `suite.only()` or `test.only()`) then run `npm test`. Or use:
```bash
npm run compile-tests && npx mocha out/test/extension.test.js --grep "test name pattern"
```

---

## Test Architecture (56 Tests, ~60% Coverage)

### Test File Structure
- `src/test/extension.test.ts`: 15 test suites, 56 passing tests
- `src/test/testUtils.ts`: mock infrastructure (MockChildProcess, MockFileSystem, MockGit) + test data

### Key Test Suites
1. **Command Registration** (2 tests): Validates all 15 commands are in constants
2. **Notifications & Messages** (3 tests): Error/success message availability
3. **Mock Infrastructure** (18 tests):
   - `MockChildProcess`: spawn() event simulation
   - `MockFileSystem`: fs.readFile/writeFile with ENOENT handling
   - `MockGit`: git stash list/pop operations
4. **Shell Command Execution** (4 tests): Success/failure/cancellation paths
5. **File Operations** (4 tests): pubspec.yaml parsing, path line detection
6. **Statistics & Error Handling** (9 tests): OperationStats tracking, error collection per module
7. **Progress & Cancellation** (4 tests): CancellationToken behavior, multiple handlers

### Coverage by Function
- `runShellCommand()`: ~70% (spawn success/failure, cancellation)
- `convertDependenciesToLocal()`: ~65% (file I/O, pubspec parsing)
- `popNamedStash()`: ~60% (git stash operations)
- Error handling paths: ~80%

### Running Specific Tests
```bash
npm test                    # All 56 tests
npm run compile-tests       # Just compile
npm run watch-tests         # Watch mode
```

---

## Key Implementation Patterns

### 1. Multi-Module Operation Pattern
Both `runProjectOperation()` and `runWorkspaceOperation()` follow this flow:
```typescript
const errors: Array<{ module: string; message: string }> = [];
let successCount = 0, errorCount = 0;
const startTime = Date.now();

await vscode.window.withProgress(..., async (progress, token) => {
  for (const item of items) {
    if (token.isCancellationRequested) break;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DanieleDituri/multi-module-flutter-tools](https://github.com/DanieleDituri/multi-module-flutter-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
