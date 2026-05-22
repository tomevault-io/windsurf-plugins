---
trigger: always_on
description: Guidelines for AI coding agents working in this repository.
---

# AGENTS.md

Guidelines for AI coding agents working in this repository.

## ⚠️ Documentation — MANDATORY

**Every code change MUST include corresponding documentation updates.** The docs app (`apps/docs`) contains MDX documentation pages that serve as the public-facing docs for the `@lsp-indexer` packages. When you modify any package (`node`, `react`, `next`, `types`, or the indexer), you MUST update the relevant docs page(s):

| Change                                     | Update                                                                            |
| ------------------------------------------ | --------------------------------------------------------------------------------- |
| New hook or server action                  | `apps/docs/src/app/docs/react/page.mdx` and/or `docs/next/page.mdx`               |
| New fetch function, parser, or key factory | `apps/docs/src/app/docs/node/page.mdx`                                            |
| New type, filter, sort, or include field   | `apps/docs/src/app/docs/node/page.mdx` and domain tables in `docs/react/page.mdx` |
| New domain (e.g., new entity type)         | All docs pages + supported domains table in `apps/docs/src/app/(home)/page.mdx`   |
| Indexer pipeline, Docker, or env changes   | `apps/docs/src/app/docs/indexer/page.mdx`                                         |
| New env variable                           | `apps/docs/.env.example` + quickstart + relevant package docs                     |
| Subscription or provider changes           | `docs/react/page.mdx` and/or `docs/next/page.mdx` + quickstart                    |

**Do NOT skip documentation. Outdated docs are worse than no docs.**

## Project Overview

LUKSO blockchain indexer — pnpm monorepo with a single indexer package:

| Package                | Purpose                                                 | Build                                      |
| ---------------------- | ------------------------------------------------------- | ------------------------------------------ |
| `@chillwhales/indexer` | Blockchain indexer with integrated ABI + entity codegen | `pnpm --filter=@chillwhales/indexer build` |

Stack: TypeScript, Subsquid framework, TypeORM, Hasura GraphQL, LUKSO LSP standards.

## Build & Dev Commands

```bash
# Build the indexer (runs codegen + tsc)
pnpm --filter=@chillwhales/indexer build

# Build all packages
pnpm build

# Format all files
pnpm format
```

**Important**: If `Follower` or other entity imports fail, rebuild the indexer — codegen generates entities from `packages/indexer/schema.graphql`.

There are no tests, no ESLint, and no git hooks configured. The only code quality tool is Prettier.

## Formatting (Prettier)

- 2-space indentation, 100 char print width
- Single quotes, trailing commas everywhere
- `prettier-plugin-organize-imports` handles import sorting automatically

## TypeScript Config

- Target: `es2020`, Module: `commonjs`
- Decorators enabled (`experimentalDecorators`, `emitDecoratorMetadata`)
- Path alias: `@/*` maps to `src/*` (e.g., `import { populateByUP } from '@/core/pluginHelpers'`)

## Code Style

### Imports — automatically sorted by Prettier

Import statements are automatically organized by `prettier-plugin-organize-imports` with no blank lines between them:

```typescript
import { v4 as uuidv4 } from 'uuid';
import { LSP5DataKeys } from '@lukso/lsp5-contracts';
import { LSP5ReceivedAsset } from '@/model';
import { Store } from '@subsquid/typeorm-store';
import { bytesToHex, Hex, hexToBytes } from 'viem';
import { mergeUpsertEntities, populateByUP } from '@/core/pluginHelpers';
import { Block, DataKeyPlugin, EntityCategory, IBatchContext, Log } from '@/core/types';
```

The plugin automatically groups and sorts imports (third-party → scoped packages → internal). Do not manually add blank lines between import groups — let the plugin handle all formatting.

Core files (`core/*.ts`) use relative imports for siblings (`./types`). All others use `@/` aliases.

### Naming Conventions

| Element             | Convention                          | Example                            |
| ------------------- | ----------------------------------- | ---------------------------------- |
| Plugin files        | `camelCase.plugin.ts`               | `lsp7Transfer.plugin.ts`           |
| Core files          | `camelCase.ts`                      | `pluginHelpers.ts`                 |
| Constants           | `UPPER_SNAKE_CASE`                  | `ENTITY_TYPE`, `LSP5_LENGTH_KEY`   |
| Variables/functions | `camelCase`                         | `batchCtx`, `extractLength`        |
| Classes             | `PascalCase`                        | `PluginRegistry`, `BatchContext`   |
| Interfaces          | `I` prefix for behavioral contracts | `IBatchContext`, `IPluginRegistry` |
| Type aliases        | `PascalCase` (no prefix)            | `VerifyFn`, `Plugin`, `Block`      |
| Enums               | `PascalCase` name + members         | `EntityCategory.UniversalProfile`  |
| Plugin objects      | `PascalCase` + `Plugin` suffix      | `LSP7TransferPlugin`               |

### Types

- Explicit return types on every function and method
- Prefer `unknown` over `any` — `any` only for TypeORM constructor signatures with `eslint-disable` comment

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chillwhales/lsp-indexer](https://github.com/chillwhales/lsp-indexer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
