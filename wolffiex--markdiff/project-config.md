---
trigger: always_on
description: Instructions for AI assistants working on this codebase.
---

# CLAUDE.md

Instructions for AI assistants working on this codebase.

## Project Overview

Markdiff is a browser-based markdown annotator. Users open local folders, view markdown files, and add annotations that are saved back to the files using Critic Markup syntax.

## Tech Stack

- **Runtime**: Bun
- **Server**: Bun.serve() with HTTPS and HTML imports
- **Frontend**: Vanilla JS, single HTML file (public/index.html)
- **Markdown**: marked.js with custom Critic Markup extensions
- **Tests**: Playwright E2E tests

## Key Files

- `server.ts` - Bun HTTPS server with HMR/console streaming
- `public/index.html` - Main application (all JS/CSS inline)
- `tests/` - Playwright E2E tests
- `tests/fixtures/test-helpers.ts` - Shared test utilities
- `tests/fixtures/mock-fs.ts` - Mock File System Access API

## Architecture

### Annotation Flow
1. User clicks block → opens annotation input
2. On save → inject `{>>comment<<}` at end of block in file
3. On load → extract annotations from Critic Markup, display in margin
4. Polling detects external changes → merge/conflict resolution

### State Management
```javascript
state = {
  annotations: {},      // blockId → comment text
  pendingReload: false, // file changed, waiting to reload
  commenting: false,    // user is mid-comment
  currentFile: { handle, lastModified, ast }
}
```

## Commands

```bash
bun --hot server.ts    # Dev server with HMR
bun run build          # Build standalone HTML
bun test:e2e           # Run Playwright tests
bun test:e2e:headed    # Tests with visible browser
bun test:e2e:debug     # Debug mode with Playwright inspector
bun test:e2e:ui        # Interactive UI mode
bun test:e2e:report    # View last test report
```

## Testing

### Running Tests

```bash
# Run all tests (headless)
bun test:e2e

# Run with visible browser (useful for seeing what's happening)
bun test:e2e:headed

# Debug mode - opens Playwright inspector, pauses on failures
bun test:e2e:debug

# Interactive UI mode - pick and run tests visually
bun test:e2e:ui

# Run a specific test file
npx playwright test tests/annotations.spec.ts

# Run tests matching a pattern
npx playwright test -g "clean merge"
```

### Test File Organization

```
tests/
  fixtures/
    mock-fs.ts          # Mock File System Access API
    test-helpers.ts     # Shared helpers and selectors
  app.spec.ts           # Basic app functionality (file tree, rendering)
  annotations.spec.ts   # Core annotation flow, merge scenarios
  save-annotations.spec.ts              # File persistence tests
  annotation-rendering-regression.spec.ts  # UI regression tests
```

### Writing New Tests

Tests use Playwright and should import from the fixtures:

```typescript
import { test, expect } from '@playwright/test';
import { injectMockFileSystem, updateMockFile } from './fixtures/mock-fs';
import {
  selectors,
  openFolderAndSelectFile,
  addAnnotation,
  expectAnnotationDisplay,
  // ... other helpers
} from './fixtures/test-helpers';

test.describe('My Feature', () => {
  test.beforeEach(async ({ page }) => {
    await injectMockFileSystem(page);  // Always inject mock FS first
    await page.goto('/');
  });

  test('does something', async ({ page }) => {
    await openFolderAndSelectFile(page, 'README.md');
    await addAnnotation(page, 'Test Documentation', 'My comment');
    await expectAnnotationDisplay(page, 'My comment');
  });
});
```

### Mock File System

The `mock-fs.ts` module simulates the File System Access API since Playwright can't use the real browser API. Key functions:

- **`injectMockFileSystem(page, mockFS?)`** - Call in `beforeEach`. Injects mock `showDirectoryPicker` into the page. Uses `defaultMockFS` if no custom structure provided.
- **`updateMockFile(page, path, content)`** - Simulates external file changes. Updates content and `lastModified` timestamp to trigger polling.
- **`defaultMockFS`** - Default test file structure with README.md, notes.md, and a guides/ subdirectory.

Custom mock FS for specific tests:

```typescript
const customFS: MockDirectory = {
  name: 'my-docs',
  files: [
    { name: 'test.md', content: '# Test\n\nContent here.' }
  ],
  directories: []
};
await injectMockFileSystem(page, customFS);
```

### Test Helpers

`test-helpers.ts` provides centralized selectors and common operations:

**Key Selectors** (use via `selectors.xxx`):
- `annotatableBlock` - Blocks that can receive annotations
- `annotationIndicator` - Margin indicator (+ or speech bubble)
- `annotationIndicatorHasComment` - Indicator showing existing annotation
- `annotationInput` - The textarea container for adding/editing
- `annotationDialog` - View dialog with Edit/Delete buttons
- `reloadBanner` - "Document changed externally" banner
- `conflictModal` / `orphanModal` - Merge conflict dialogs

**Common Operations**:
- `openFolderAndSelectFile(page, 'file.md')` - Opens folder and selects a file
- `addAnnotation(page, blockText, annotationText)` - Adds annotation to block containing text
- `openAnnotationInput(page, blockText)` - Opens input without saving
- `expectAnnotationDisplay(page, text)` - Asserts annotation exists
- `expectReloadBanner(page, visible)` - Checks banner visibility
- `handleConflictModal(page, 'apply'|'discard')` - Handles conflict dialog

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wolffiex) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
