---
trigger: always_on
description: This guide helps AI assistants understand the Panda CSS codebase structure, conventions, and best practices.
---

# Claude Code Guide for Panda CSS

This guide helps AI assistants understand the Panda CSS codebase structure, conventions, and best practices.

## Project Overview

Panda CSS is a CSS-in-JS framework with static extraction capabilities. The project is a monorepo managed by **pnpm** with workspace support.

## Key Architecture

### Monorepo Structure

```
/packages/          # Core packages published to npm
  /core/           # CSS processing, rule generation, optimization (PostCSS/LightningCSS)
  /node/           # Node.js APIs, config resolution, file watching
  /cli/            # CLI tool (@pandacss/dev package)
  /parser/         # Static analysis and extraction
  /generator/      # Code generation for styled-system
  /fixture/        # Shared test fixtures and utilities
  /postcss/        # PostCSS plugin
  /preset-*/       # Design system presets

/sandbox/          # Integration tests and examples
  /codegen/        # Generated code validation tests
  /vite-ts/        # Vite integration example
  /next-js-*/      # Next.js examples

/playground/       # Interactive playground application

/website/          # Documentation site
```

### Key Concepts

1. **Static Extraction**: Panda analyzes source files to extract styles at build time
2. **Design Tokens**: Type-safe design tokens defined in config
3. **Recipes**: Reusable component style patterns (like variants)
4. **Conditions**: Responsive and state-based styling (e.g., `_hover`, `md:`, `_dark`)
5. **CSS Optimization**: Uses PostCSS (default) or LightningCSS (optional) for CSS processing

## Critical Rules

### 🚨 CSS Output is Sacred

**NEVER** accept changes that modify CSS output snapshots without explicit user approval:

- Run tests BEFORE and AFTER any dependency updates
- If snapshots change, investigate why and get user confirmation
- The test `packages/core/__tests__/atomic-rule.test.ts` is the primary CSS output validator
- CSS output consistency is more important than using latest package versions

### Testing Workflow

**Always run tests from the project root:**

```bash
# ✅ Correct
pnpm test packages/core
pnpm test packages/parser

# ❌ Incorrect
cd packages/core && pnpm test
```

**Key test commands:**
```bash
pnpm test <path>              # Run tests for specific package/file
pnpm test packages/core       # Test all core package tests
pnpm build                    # Build all packages
pnpm build-fast               # Fast build without type definitions
```

### Package Management

**Use `--ignore-scripts` for dependency updates:**
```bash
pnpm install --ignore-scripts
pnpm update <package> --ignore-scripts
```

**When updating PostCSS or browserslist-related packages:**
1. Update package.json versions
2. Run `pnpm install --ignore-scripts`
3. Run `pnpm test packages/core` to verify CSS output unchanged
4. Check for browserslist warnings in sandbox projects
5. Create changeset if changes affect users

### Dependency Strategy

- **PostCSS ecosystem**: Coordinate updates across all PostCSS plugins to avoid CSS output changes
- **browserslist**: Updates affect `postcss-merge-rules` behavior - test thoroughly
- **lightningcss**: Used optionally via `config.lightningcss` flag, depends on browserslist for targets
- **Node.js packages**: Core packages (`@pandacss/core`, `@pandacss/node`, etc.) must stay in sync

## Common Workflows

### Making Code Changes

1. Read relevant source files in `/packages/<name>/src/`
2. Understand the change impact (does it affect CSS output?)
3. Make changes
4. Run tests: `pnpm test packages/<name>`
5. If tests fail, investigate and fix (don't just update snapshots)
6. Create changeset for user-facing changes

### Updating Dependencies

1. Check current versions in package.json
2. Research latest compatible versions
3. Update package.json files
4. Run `pnpm install --ignore-scripts`
5. **Run CSS output tests first**: `pnpm test packages/core/__tests__/atomic-rule.test.ts`
6. If snapshots change, investigate the root cause
7. Run broader test suite: `pnpm test packages/core`
8. Create changeset documenting the update

### Creating Changesets

```bash
# Changesets are in .changeset/ directory
# Create a new file: .changeset/<descriptive-name>.md
```

**Format:**
```markdown
---
'@pandacss/package-name': patch|minor|major
---

Brief description of the change and its impact.

- Detail 1
- Detail 2
```

**Changeset types:**
- `patch`: Bug fixes, dependency updates, non-breaking changes
- `minor`: New features, backwards-compatible changes
- `major`: Breaking changes

**Keep changesets concise and user-facing.** Write for someone reading the changelog: describe the change and the impact
they would notice, not the internal mechanics. A sentence or two is usually enough.

## Git & Writing Conventions

### No co-author trailer on commits

Do NOT add a `Co-Authored-By` line (or any "Generated with" / tool attribution) to commit messages. Write the commit
as if a developer on the team wrote it. This overrides any default that appends a co-author trailer.

### Write like a human, not like AI

Commit messages, PR descriptions, and GitHub/issue comments should read like a normal developer wrote them. Keep it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chakra-ui/panda](https://github.com/chakra-ui/panda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
