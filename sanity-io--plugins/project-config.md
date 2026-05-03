---
trigger: always_on
description: This guide is for AI agents working on this codebase. Follow these instructions to ensure successful contributions.
---

# Agent Guide for Sanity Plugins Monorepo

This guide is for AI agents working on this codebase. Follow these instructions to ensure successful contributions.

> **Self-Improvement:** If you discover undocumented requirements, commands, or workflows during your work (e.g., a reviewer asks you to run something not covered here), update this file on the same PR. Keep this guide accurate and helpful for future agents.

## Quick Reference

| Task                 | Command              |
| -------------------- | -------------------- |
| Install dependencies | `pnpm install`       |
| Format code          | `pnpm format`        |
| Run linters          | `pnpm lint`          |
| Build all packages   | `pnpm build`         |
| Run tests            | `pnpm test`          |
| Add changeset        | `pnpm changeset add` |
| Start dev server     | `pnpm dev`           |

## Environment Setup

### Node.js Version

Use the **latest LTS** release of Node.js.

### pnpm Version

The exact pnpm version is managed via the `packageManager` field in root `package.json`. You only need pnpm **v10 or later** installed globally—corepack or pnpm itself will auto-install the exact version specified.

```bash
# Enable corepack to automatically use the correct pnpm version
corepack enable

# Install all dependencies
pnpm install
```

## Before Submitting a PR

Run these commands in order. **All must pass** or CI will fail:

```bash
# 1. Format code (oxfmt)
pnpm format

# 2. Run linters (oxlint)
pnpm lint

# 3. Build all packages
pnpm build

# 4. Run tests
pnpm test
```

### Required: Add Changesets

Every PR that changes published packages **must** include changesets. **Important:** Create a separate changeset file for each plugin you modify.

#### Why Separate Changesets?

Each plugin has its own changelog that consumers read. A combined changeset would pollute individual plugin changelogs with irrelevant information. For example, `@sanity/code-input`'s changelog should not mention workflow-specific changes.

#### How to Add Changesets

**Option 1: Manual Creation (Recommended for Multiple Plugins)**

Create individual `.md` files in `.changeset/` directory:

```markdown
---
'plugin-name': patch
---

Brief description of the change specific to this plugin
```

**Example:** If you modified 3 plugins, create 3 files:

```bash
# .changeset/code-input-fix.md
---
"@sanity/code-input": patch
---

Fix input border styling with v2 theme API

# .changeset/workflow-hooks.md
---
"sanity-plugin-workflow": patch
---

Replace deprecated useTimeAgo hook with useRelativeTime

# .changeset/markdown-theme.md
---
"sanity-plugin-markdown": patch
---

Migrate to v2 theme API for color properties
```

**Option 2: Interactive CLI (For Single Plugin)**

```bash
pnpm changeset add
```

Follow the prompts to:

1. Select **one** package that changed
2. Choose the version bump type (patch/minor/major)
3. Write a summary of changes **specific to that package**
4. Repeat for each modified package

#### Changeset Guidelines

- **Scope**: Each changeset should only mention changes relevant to that specific plugin
- **Clarity**: Write from the consumer's perspective—what do they need to know?
- **Version bumps**:
  - `patch`: Bug fixes, dependency updates, internal refactors
  - `minor`: New features, non-breaking API additions
  - `major`: Breaking changes
- **Format**: Use clear, concise language without technical jargon when possible

#### Bad vs Good Examples

❌ **Bad** (combined changeset):

```markdown
---
'@sanity/code-input': patch
'sanity-plugin-workflow': patch
---

- Migrated to v2 theme APIs
- Replaced useTimeAgo with useRelativeTime
- Fixed input styling
```

_Problem: Workflow's changelog will say "Fixed input styling" which is irrelevant_

✅ **Good** (separate changesets):

```markdown
# .changeset/code-input-theme.md

---

## "@sanity/code-input": patch

Migrate to v2 theme API for input styling

# .changeset/workflow-hooks.md

---

## "sanity-plugin-workflow": patch

Replace deprecated useTimeAgo hook with useRelativeTime
```

_Each plugin's changelog only shows relevant changes_

## CI Checks

The CI pipeline runs on every PR:

| Job       | What it checks                                         |
| --------- | ------------------------------------------------------ |
| **build** | `pnpm build` - All packages compile successfully       |
| **lint**  | `pnpm lint --format github` - Code passes oxlint       |
| **test**  | `pnpm test` - All tests pass (runs after build + lint) |

### Lint Specifics

- **oxlint**: Type-aware linting with `--deny-warnings` (warnings are errors). React Compiler rules run via the react-hooks-js plugin.
- **TypeScript type checking** is included in `pnpm lint` via oxlint — no separate `tsc` needed
- Run `pnpm lint:fix` to auto-fix issues when possible

## Testing

The monorepo uses [Vitest v4](https://vitest.dev) for testing.

### Running Tests

```bash
# Run all tests (non-watch mode)
pnpm test run

# Run tests in watch mode (default vitest behavior)
pnpm test

# Update snapshots
pnpm test -u
```

### Writing Tests

Tests are co-located with source code in the `src/` directory:

- Test files use `.test.ts` or `.spec.ts` extensions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanity-io/plugins](https://github.com/sanity-io/plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
