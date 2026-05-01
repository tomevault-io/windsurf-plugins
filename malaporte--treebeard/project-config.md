---
trigger: always_on
description: Treebeard is an **Electrobun** desktop app for managing Git worktrees across repositories,
---

# AGENTS.md — Treebeard

## Project Overview

Treebeard is an **Electrobun** desktop app for managing Git worktrees across repositories,
with Jira issue badges, GitHub PR/CI status, and quick-launch buttons for VS Code
and Ghostty.

**Stack:** TypeScript (strict), React 18, Mantine v7, Electrobun, Bun

**Structure:**
- `packages/treebeard/` — The Electrobun desktop app package
- `packages/treebeard/src/bun/` — Main process entry (`index.ts`), shared types (`../shared/types.ts`)
- `packages/treebeard/src/bun/services/` — Backend services: `git.ts`, `github.ts`, `jira.ts`, `launcher.ts`, `config.ts`, `dependencies.ts`, `shell-env.ts`
- `packages/treebeard/src/shared/` — Shared types and RPC schema (`types.ts`, `rpc-types.ts`)
- `packages/treebeard/src/` — Renderer process (React app)
- `packages/treebeard/src/components/` — Flat directory of single-purpose React components
- `packages/treebeard/src/hooks/` — Custom hooks (`useWorktrees`, `usePR`, `useJiraIssue`, `useConfig`)

## Build / Run Commands

| Command            | Description                                      |
| ------------------ | ------------------------------------------------ |
| `bun run dev`         | Start in development mode with hot-reload     |
| `bun run build`       | Production build via Electrobun               |
| `bun run typecheck`   | Type-check only (`tsc --noEmit`)              |
| `bun run test`        | Run Vitest test suite                         |

Vitest is configured for the desktop/bun codebase.

For targeted tests:
```bash
bun vitest run path/to/file.test.ts          # single test file
bun vitest run -t "test name pattern"         # single test by name
```

## Releasing

Use the `release` skill to walk through the full release process:

```
/release
```

The skill will read the current version, inspect commits since the last tag to
recommend a semver bump, update `packages/treebeard/package.json`, commit, tag,
and push to trigger the CI/CD release pipeline.

### Manual steps (for reference)

1. Bump the version in `packages/treebeard/package.json`
2. Commit the version bump (e.g., `chore: bump version to X.Y.Z`)
3. Create a git tag: `git tag vX.Y.Z`
4. Push the commit and tag: `git push && git push --tags`

The tag push triggers the CI/CD pipeline that builds and publishes the release.

## Code Style

### Imports

Three groups, no blank lines between them:
1. Node.js stdlib (always use `node:` prefix: `import path from 'node:path'`)
2. Third-party packages (`@mantine/core`, `@tabler/icons-react`)
3. Local/relative imports (`./components/...`, `../hooks/...`)

Type-only imports use `import type` on a separate line, placed last in the import block:
```ts
import { execFile } from 'node:child_process'
import { Card, Text, Group } from '@mantine/core'
import { JiraBadge } from './JiraBadge'
import type { Worktree } from '../../shared/types'
```

### Naming Conventions

| Kind                | Convention       | Examples                                         |
| ------------------- | ---------------- | ------------------------------------------------ |
| Component files     | PascalCase       | `WorktreeCard.tsx`, `PRBadge.tsx`                |
| Hook files          | camelCase        | `useWorktrees.ts`, `usePR.ts`                   |
| Service files       | camelCase        | `git.ts`, `github.ts`                           |
| React components    | PascalCase       | `function WorktreeCard()`                        |
| Hooks               | `use` prefix     | `function useJiraIssue()`                        |
| Event handlers      | `handle` prefix  | `handleSubmit`, `handleBrowse`                   |
| Functions/variables | camelCase        | `getWorktrees`, `worktreePath`                   |
| Interfaces/types    | PascalCase       | `Worktree`, `PRInfo`, `AppConfig`                |
| Props interfaces    | `*Props` suffix  | `WorktreeCardProps`, `PRBadgeProps`              |
| Constants           | UPPER_SNAKE_CASE | `MAIN_BRANCH_NAMES`, `JIRA_KEY_REGEX`           |

### Types

- Use `interface` for all object shapes. Reserve `type` for aliases and unions only.
- Explicit return type annotations on **exported service functions** (`Promise<Worktree[]>`).
- Inferred return types on React components and hooks.
- Use `import type` consistently for type-only imports — never mix into value imports.
- Prefer inline union literals over enums: `'OPEN' | 'CLOSED' | 'MERGED'`.

### Functions

- **Components and hooks:** Always use `function` declarations, never arrow functions.
  ```ts
  export function WorktreeCard({ worktree, repoPath }: WorktreeCardProps) {
  ```
- **Callbacks and handlers:** Use arrow functions.
  ```ts
  const handleVSCode = async () => { ... }
  worktrees.filter((wt) => wt.branch.includes(query))
  ```

### Exports

- **Named exports only.** No default exports (sole exception: root `App.tsx`).
- **No barrel files.** Every import targets the specific file directly.

### Error Handling

- **Services (bun side):** Catch errors silently and return `null` for non-critical failures.
  Use empty `catch` blocks (no error variable) when the error is not inspected:
  ```ts
  catch {
    return null
  }
  ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [malaporte/treebeard](https://github.com/malaporte/treebeard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
