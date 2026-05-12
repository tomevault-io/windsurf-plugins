---
trigger: always_on
description: Proofly is a privacy-first Chrome extension for proofreading that uses Chrome's Built-in AI API for on-device text correction.
---

# Proofly Development Guide

Proofly is a privacy-first Chrome extension for proofreading that uses Chrome's Built-in AI API for on-device text correction.

## 🎯 Core Principles

1. **Privacy**: Zero data leaves the user's device
2. **Performance**: Lightweight scripts, lazy loading, minimal overhead
3. **Non-invasiveness**: Zero dependencies, Shadow DOM isolation, no code pollution
4. **Accessibility**: Free, open-source, works offline

## 🏗️ Architecture

### Technology Stack

- **TypeScript**: Strict mode, comprehensive type coverage
  - **Type Definitions**: Add external type packages to `tsconfig.json` `types` array, NOT via reference imports
  - Example: `"types": ["vite/client", "chrome", "@types/dom-chromium-ai"]`
- **Vite + CRXJS**: Modern build pipeline
- **Web Components**: Shadow DOM for UI isolation
- **Vanilla JS**: No frameworks—keep bundle size minimal
- **Functional Programming**: Pure functions, composition, no side effects
- **Dependency Injection**: Services as modules/parameters for testing
- **Design Tokens**: CSS custom properties for theming

### Key Decisions

1. **Modular & Extensible**: Loosely coupled modules with single responsibilities
2. **Functional Core**: Pure functions, easy to test and compose
3. **Dependency Injection**: Never use singletons or global state
4. **Shadow DOM Everywhere**: All UI components MUST use Shadow DOM
5. **Lazy Loading**: Heavy components load only on user interaction
6. **Content Script Minimalism**: Initial injection <5KB, dynamic imports
7. **Zero Dependencies**: Pure vanilla TypeScript only
8. **Design Token System**: CSS custom properties for consistency

## 🔧 Development Guidelines

### Logging Guidelines

**IMPORTANT**:

- **ALWAYS use `logger.info()` for debugging** instead of console.log
- **Use `logger.warn()` for warnings** instead of console.warn
- **Use `logger.error()` for errors** instead of console.error
- **NEVER add 'Proofly' prefix to log messages** - the logger handles context automatically
- The logger is available at `src/services/logger.ts` and should be imported in all modules
- Logger automatically includes context (background, content, options, etc.) and session information

**Example**:

```typescript
import { logger } from '../services/logger.ts';

// ✅ Good: Use logger for debugging
logger.info('Processing proofread request', { textLength: text.length });
logger.warn('Model not ready, queueing request');
logger.error('Failed to proofread', { error });

// ❌ Bad: Using console directly
console.log('Processing request');
console.warn('Model not ready');
console.error('Failed to proofread');

// ❌ Bad: Adding prefix manually
logger.info('Proofly - Processing request'); // Don't do this!
```

### Build Verification

**IMPORTANT**:

- **DO NOT run `npm run build` manually** unless explicitly instructed by the user
- The `npm run dev` script is already running in a separate terminal and automatically builds the extension on file changes
- The dev script watches for changes and rebuilds automatically, providing faster feedback during development
- Only run `npm run build` if specifically requested by the user for production builds or troubleshooting

### Code Formatting

**IMPORTANT**:

- **ALWAYS run `npm run format` after making code changes** to ensure consistent code style
- This formats all TypeScript, JavaScript, JSON, CSS, Markdown, and HTML files according to the project's Prettier configuration
- Formatting is not automatic - you must run the format command manually after changes
- Use `npm run format:check` to verify formatting without making changes (useful for validation)

**Example Workflow**:

```bash
# 1. Make code changes
# 2. Format the code
npm run format

# 3. Verify changes (dev script auto-rebuilds)
# 4. Test the changes
```

### Iteration Testing Checklist

- On every feature/fix iteration run `npm run typecheck` followed by `npm test` to catch regressions early.
- Always inspect the full console output of `npm run test` (pipe to a log if needed) instead of relying solely on the exit status.
- When adding a brand-new e2e scenario, temporarily focus it with `test.only(...)`, run `npm run test:e2e`, and remove the focus flag before committing.

### Standard Testing Workflow for Content Scripts

**FOR EVERY CODE CHANGE**, follow this complete verification workflow:

#### 1. Make Changes

- Implement the requested changes in the codebase
- Dev script auto-rebuilds the extension

#### 2. Open Test Environment

- Ensure test server is running: `python3 -m http.server 8080` (in project root)
- Navigate to http://localhost:8080/test.html (local test page with input, textarea, and contenteditable)
- Or use any other test page appropriate for the feature

#### 3. Type Test Input

- Enter text with intentional errors, issues, or content that exercises the feature
- Simulate real user interactions

#### 4. Verify Changes (4-Step Verification)

**A. Element Inspection via Page Snapshot**

```typescript
mcp__chrome - devtools__take_snapshot({ verbose: false });
```

- Purpose: Inspect DOM structure, verify highlights/corrections are applied
- Check: Element attributes, classes, data attributes, Shadow DOM content

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onderceylan/proofly](https://github.com/onderceylan/proofly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
