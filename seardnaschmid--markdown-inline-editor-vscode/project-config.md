---
trigger: always_on
description: This document provides essential context and guidelines for AI agents working on this VS Code extension project. Follow these instructions to ensure your contributions align with project standards.
---

# AI Agent Guide for Markdown Inline Editor

This document provides essential context and guidelines for AI agents working on this VS Code extension project. Follow these instructions to ensure your contributions align with project standards.

## Quick Start Checklist

Before making changes:
1. ✅ Read this file completely
2. ✅ Understand the project structure (see below)
3. ✅ Run `npm run validate` to ensure current state is clean
4. ✅ Identify the correct files to modify in `/src/`
5. ✅ Write/update tests in corresponding `__tests__/` directories
6. ✅ Verify changes with `npm run validate` before committing

## Project Context

**What This Project Does:**
- VS Code extension that renders markdown syntax inline (WYSIWYG-style)
- Uses VS Code TextEditorDecorationType to hide/show markdown syntax
- Parses markdown using remark and applies visual decorations
- Supports links, images, headings, lists, code blocks, and more

**Tech Stack:**
- TypeScript (strict mode)
- VS Code Extension API
- [remark](https://github.com/remarkjs/remark) for markdown parsing
- Jest for testing
- esbuild for bundling

## Project Structure

### Source Code (`/src/`)

**Core Files:**
- `extension.ts` - Extension entry point, activation, command registration
- `config.ts` - Centralized configuration access (VS Code settings)
- `parser.ts` - Main markdown parser, converts AST to decoration ranges
- `parser-remark.ts` - Remark parser setup and utilities
- `decorations.ts` - Decoration type factories (transparent, faint, etc.)
- `decorator.ts` - Decoration orchestration, applies decorations to editors

**Specialized Modules:**
- `diff-context.ts` - Detects diff views and applies policies
- `link-targets.ts` - Resolves link/image URLs (relative, absolute, workspace)
- `markdown-parse-cache.ts` - Caching layer for parsed markdown (performance critical)
- `position-mapping.ts` - Handles CRLF/LF normalization for position calculations

**Feature Modules:**
- `link-provider.ts` - Makes links clickable (DocumentLinkProvider)
- `link-hover-provider.ts` - Shows link URLs on hover
- `image-hover-provider.ts` - Shows image previews on hover
- `link-click-handler.ts` - Handles single-click navigation

**Decoration System:**
- `decorator/decoration-type-registry.ts` - Manages decoration type lifecycle
- `decorator/visibility-model.ts` - 3-state filtering (Rendered/Ghost/Raw)
- `decorator/checkbox-toggle.ts` - Handles checkbox clicks
- `decorator/decoration-categories.ts` - Categorizes decoration types

**Test Directories:**
- Each module has a corresponding `__tests__/` directory
- Test files use `.test.ts` extension
- Follow naming: `module-name.test.ts`

### Other Directories

- `/dist/` - Compiled output (DO NOT EDIT - generated files)
- `/docs/` - Documentation, feature specs, analysis
- `/scripts/` - Build and release automation
- `/assets/` - Icons and static files

## Available Commands

**Build & Development:**
```bash
npm run compile    # TypeScript compilation only
npm run build   # Full build (compile + bundle + package)
npm run clean   # Remove build artifacts
npm run package # Package extension as .vsix
```

**Testing:**
```bash
npm test              # Run all tests
npm run test:watch     # Run tests in watch mode
npm run test:coverage # Generate coverage report
npm run test:crlf     # Run CRLF-specific tests
```

**Validation:**
```bash
npm run lint          # Run ESLint
npm run lint:docs     # Validate feature file structure
npm run validate      # Run ALL checks (lint:docs + test + build)
```

**Release:**
```bash
npm run release       # Automated release (see Release section)
```

## Critical Rules for AI Agents

### 1. File Modification Boundaries

**✅ DO:**
- Modify files in `/src/` only
- Edit test files in `*/__tests__/` directories
- Update documentation in `/docs/` when adding features
- Modify `package.json` only for dependencies or scripts

**❌ DO NOT:**
- Edit files in `/dist/` (generated, will be overwritten)
- Modify `.vscodeignore` unless explicitly asked
- Change build configuration without understanding impact
- Edit `CHANGELOG.md` manually (auto-generated)

### 2. Performance Requirements

**Cache Usage:**
- ALWAYS use `markdown-parse-cache.ts` for parsing
- NEVER parse the entire document on selection change
- Cache results and reuse when possible

**Large File Handling:**
- Handle malformed markdown gracefully (don't crash)
- Test with large files (>10k lines) if making parser changes
- Use efficient algorithms (avoid O(n²) operations)

### 3. Testing Requirements

**Before Committing:**
1. Write tests for new functionality
2. Update existing tests if behavior changes
3. Run `npm test` and ensure all pass
4. Check test coverage if adding new modules

**Test Structure:**
- Place tests in `src/module-name/__tests__/module-name.test.ts`
- Use descriptive test names: `describe('feature', () => { it('should do X', ...) })`
- Test edge cases: empty input, malformed markdown, large files
- Mock VS Code API when needed (see existing tests for patterns)

**Current Test Coverage:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeardnaSchmid/markdown-inline-editor-vscode](https://github.com/SeardnaSchmid/markdown-inline-editor-vscode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
