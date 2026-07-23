---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

<!-- nx configuration start-->
<!-- Leave the start & end comments to automatically receive updates. -->

# General Guidelines for working with Nx

- For navigating/exploring the workspace, invoke the `nx-workspace` skill first - it has patterns for querying projects, targets, and dependencies
- When running tasks (for example build, lint, test, e2e, etc.), always prefer running the task through `nx` (i.e. `nx run`, `nx run-many`, `nx affected`) instead of using the underlying tooling directly
- Prefix nx commands with the workspace's package manager (e.g., `pnpm nx build`, `npm exec nx test`) - avoids using globally installed CLI
- You have access to the Nx MCP server and its tools, use them to help the user
- For Nx plugin best practices, check `node_modules/@nx/<plugin>/PLUGIN.md`. Not all plugins have this file - proceed without it if unavailable.
- NEVER guess CLI flags - always check nx_docs or `--help` first when unsure

## Scaffolding & Generators

- For scaffolding tasks (creating apps, libs, project structure, setup), ALWAYS invoke the `nx-generate` skill FIRST before exploring or calling MCP tools

## When to use nx_docs

- USE for: advanced config options, unfamiliar flags, migration guides, plugin configuration, edge cases
- DON'T USE for: basic generator syntax (`nx g @nx/react:app`), standard commands, things you already know
- The `nx-generate` skill handles generator discovery internally - don't call nx_docs just to look up generator syntax

<!-- nx configuration end-->

# Repository overview

Nx monorepo (yarn 4, berry) that publishes the `json-difference` library — a path-based JSON delta computer — plus ancillary projects that consume it.

| Project | Path | Type | Notes |
|---|---|---|---|
| `json-difference` | `libs/json-difference` | library (published) | Core lib, ESM/CJS + standalone browser bundle |
| `json-difference-cli` | `libs/json-difference-cli` | library (published) | `jd` CLI wrapper (yargs) |
| `mcp-json-diff` | `tools/mcp-json-diff` | application | MCP server exposing the lib via stdio |
| `playground` | `apps/playground` | application | React + Vite demo deployed to GitHub Pages |
| `example` | `apps/example` | application | Runnable Node usage scripts (`simple.ts`, `stress.ts`) |

Node >=18.17. Package manager is yarn — **do not** use npm/pnpm here (releases use `yarn dlx nx`).

# Common commands

Prefer `yarn nx` over project-local tooling. Key targets:

```sh
yarn nx test json-difference                 # Jest, ts-jest, finds **/*.spec.ts
yarn nx test json-difference --testFile=get-diff.spec.ts   # single spec
yarn nx build json-difference                # vite build (ESM/CJS via vite.config.mts)
yarn nx run json-difference:build-browser    # standalone browser bundle (vite.config.browser.mts)
yarn nx run json-difference:build-to-s3      # build-browser + prepare-to-s3.ts (versioned bundle)
yarn nx lint json-difference                 # ESLint (maxWarnings: 0)
yarn nx run-many --target=type-check         # tsc --noEmit across all projects

yarn nx serve playground                     # React dev server
yarn nx run example:test                     # runs simple.ts + stress.ts via tsx
yarn nx run mcp-json-diff:serve              # run MCP server directly (stdio)
yarn nx run mcp-json-diff:test-browser       # open @modelcontextprotocol/inspector UI
yarn nx run json-difference-cli:install-local # build CLI and `npm install -g` it

yarn graph                                   # full dep graph; yarn graph:affected for subset
yarn release[:dry-run]                       # nx release: version + changelog + publish (do not run casually)
```

The `claude` script (`yarn claude`) builds the MCP server then launches Claude Code with it available — `.mcp.json` points at `tools/mcp-json-diff/bin/src/index.js`, so the MCP server must be built before it will respond.

# Core library architecture

The library is a 4-function pipeline. Understand `getStructPaths` first — everything else operates on its output.

1. **`getStructPaths(struct, isLodashLike)`** flattens any JSON into a `Record<path, leafValue>`. Non-leaf nodes get sentinel values: `"@{}"` for objects, `"@[]"` for arrays. The synthetic `__root__` key records the top-level container type. `__start__` is an internal recursion sentinel — never leaks to output.
2. **`getPathsDiff(A, B)`** returns `[path, value]` tuples present in A but missing in B. `getDiff` calls it twice (A−B → `removed`, B−A → `added`).
3. **`getEditedPaths(oldPaths, newPaths)`** returns `[path, oldValue, newValue]` for paths present in both with changed values.
4. **`sanitizeDelta`** (`helpers/sanitize-delta.ts`) strips/rewrites the `@{}` / `@[]` sentinels so they never appear in user-visible output. Any new internal sentinel must be handled here or it will leak.

`getDiff` is the thin public composition of these four. JSON-string inputs are `JSON.parse`-ed up front, so the core routines always see real objects/arrays.

**Path formats** are mutually exclusive modes set by `isLodashLike`:
- default (slash): `a/b/0[]` — array indices marked by trailing `[]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukascivil/json-difference](https://github.com/lukascivil/json-difference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
