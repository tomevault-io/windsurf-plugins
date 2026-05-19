---
trigger: always_on
description: Open source i18n libraries for React, Next.js, and more. Repo: `generaltranslation/gt`.
---

# General Translation Monorepo

Open source i18n libraries for React, Next.js, and more. Repo: `generaltranslation/gt`.

## Monorepo Structure

- **Package manager:** pnpm (v10.20.0)
- **Build system:** Turbo (`turbo.json` defines task graph)
- **Releases:** Changesets (`pnpm changeset` to add, `changeset publish` to release)
- **Testing:** Vitest across all packages
- **Linting:** oxlint + oxfmt (enforced via lefthook pre-commit)
- **License:** FSL-1.1-ALv2

## Commands

```sh
pnpm build          # Build all packages (turbo, cached)
pnpm test           # Test all packages
pnpm lint           # Lint all packages
pnpm lint:fix       # Lint + auto-fix
pnpm format         # Check formatting with oxfmt
pnpm format:fix     # Format everything with oxfmt
pnpm changeset      # Create a changeset for a new release
pnpm version-packages  # Apply changesets to bump versions
pnpm release        # Publish packages (changeset publish)
```

Per-package commands: `pnpm --filter <pkg> <script>` (e.g., `pnpm --filter gt test`).

Turbo tasks: `build`, `test`, `lint`, `lint:fix`, `format`, `format:fix`, `transpile`, `build:clean`, `build:release`, `bench`.

## pnpm Worktrees

- pnpm's global virtual store is enabled via `enableGlobalVirtualStore: true` in `pnpm-workspace.yaml` so git worktrees share the pnpm store while keeping isolated `node_modules`. Hoisting is disabled with `hoist: false` because pnpm's hoisted dependency workaround relies on `NODE_PATH`, which does not work for ESM.
- After creating a new worktree, run `pnpm install` inside it.
- If pnpm prompts to recreate `node_modules` in a non-interactive shell, use `pnpm install --force`. Do not use `CI=true` for this because pnpm disables the global virtual store in CI mode.
- Treat missing module/type errors after install as real missing direct dependencies. Add the dependency to the package that imports or uses it, not to a sibling package or the workspace root just to make hoisting work.

## Key Packages

| Package                                 | Path                         | Description                                                       |
| --------------------------------------- | ---------------------------- | ----------------------------------------------------------------- |
| `generaltranslation`                    | `packages/core`              | Pure JS, i18n helpers and API client                              |
| `gt-i18n`                               | `packages/i18n`              | Pure JS i18n runtime                                              |
| `gt-react`                              | `packages/react`             | React i18n with `<T>` component, hooks, providers                 |
| `@generaltranslation/react-core`        | `packages/react-core`        | Pure React i18n primitives (no framework deps)                    |
| `gt-next`                               | `packages/next`              | Next.js integration (server/client split, SWC plugin, middleware) |
| `gt-node`                               | `packages/node`              | Node.js backend translation utilities                             |
| `gt-react-native`                       | `packages/react-native`      | React Native i18n with native module support                      |
| `gt-tanstack-start`                     | `packages/tanstack-start`    | TanStack Start integration                                        |
| `gt-sanity`                             | `packages/sanity`            | Sanity CMS plugin                                                 |
| `@generaltranslation/compiler`          | `packages/compiler`          | Build plugin (webpack, Vite, Rollup, esbuild) via unplugin        |
| `gt`                                    | `packages/cli`               | Main CLI tool (`npx gt`)                                          |
| `gtx-cli`                               | `packages/gtx-cli`           | Wrapper CLI for gt (backward compatibility)                       |
| `locadex`                               | `packages/locadex`           | AI agent for i18n with MCP support                                |
| `@generaltranslation/mcp`               | `packages/mcp`               | MCP server for AI tool integration                                |
| `@generaltranslation/react-core-linter` | `packages/react-core-linter` | ESLint plugin for react-core                                      |
| `@generaltranslation/gt-next-lint`      | `packages/next-lint`         | ESLint plugin for gt-next                                         |
| `gt-remark`                             | `packages/remark`            | Remark plugin for MDX escaping                                    |
| `@generaltranslation/python-extractor`  | `packages/python-extractor`  | Python source extraction (tree-sitter)                            |

## Code Conventions

- TypeScript everywhere. Strict mode.
- oxfmt: single quotes, 2-space indent, trailing commas (es5), semicolons, LF line endings.
- ESLint: `@typescript-eslint` rules, unused vars prefixed with `_`, no explicit `any` (warn).
- Prefer `const` over `let`. Never `var`.
- Test files: `*.test.ts` / `*.spec.ts` using Vitest.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [generaltranslation/gt](https://github.com/generaltranslation/gt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
