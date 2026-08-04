---
trigger: always_on
description: This file provides guidance to OpenAI Codex and other AI coding agents when working in this repository. See CLAUDE.md for the full reference — this file contains the same essential guidance.
---

# AGENTS.md

This file provides guidance to OpenAI Codex and other AI coding agents when working in this repository. See CLAUDE.md for the full reference — this file contains the same essential guidance.

## Commands

```bash
pnpm build              # dev build (all packages)
pnpm build:prod         # production build (minified, required before publish)
pnpm test               # run all tests
pnpm typecheck          # type-check all packages
pnpm prerelease:check   # smoke + validate before publishing
pnpm release:beta       # full beta publish pipeline
pnpm clean              # clear turbo and node_modules caches
```

Run tests for a single package:

```bash
pnpm --filter @ngcompass/engine vitest run
pnpm --filter @ngcompass/rules vitest run src/rules/reactivity/rxjs-no-subscribe-in-component.rule.test.ts
```

## Monorepo Structure

Turborepo + pnpm workspaces. All packages live in `packages/`. Root `package.json` is `private: true` and is never published.

| Package              | Purpose                                                             |
| -------------------- | ------------------------------------------------------------------- |
| `packages/cli`       | Binary (`ngcompass`), command orchestration                         |
| `packages/config`    | Config discovery, validation, normalization                         |
| `packages/scanner`   | File discovery (git/glob), file-list cache                          |
| `packages/rules`     | All built-in rules, presets, RuleRegistry                           |
| `packages/planner`   | Task graph, content-addressed task IDs, incremental filtering       |
| `packages/engine`    | Single-pass AST execution, worker pool, type-aware chunking         |
| `packages/ast`       | Oxc TS parser, Angular HTML parser, CSS parser, stream types        |
| `packages/cache`     | Multi-layer cache (config, file, plan, result, analysis, meta)      |
| `packages/reporters` | Console, JSON, SARIF, HTML reporters                                |
| `packages/common`    | Shared types: `RuleContext`, `RuleResult`, `AnalysisResult`, `Task` |

`packages/site` is docs only — excluded from build and publish.

## Architecture in One Paragraph

The CLI orchestrates a strict pipeline: load config → discover files → resolve rules → build execution plan → run analysis → emit reports. The **planner** converts files + rules into content-addressed tasks (`taskId` = hash of file content + rule options). Warm runs skip tasks whose `taskId` already exists in the result cache — sometimes the entire analysis is skipped via a full-analysis cache hit. The **engine** splits tasks into syntax-only (worker pool, parallel) and type-aware (main thread, chunked TypeScript Programs). Rules are passive stream observers registered in `RuleRegistry`; they never do their own AST traversal.

Full architecture reference: `docs/architecture.md`.

## Adding a Rule

1. Create `packages/rules/src/rules/<domain>/<rule-name>.rule.ts`
2. Use a factory from `@ngcompass/engine` matching the stream type you need:
   `createComponentRule`, `createTemplateExpressionRule`, `createCallExpressionRule`, etc.
3. Declare `RuleMetadata` — set `dependencyType` to `syntax`, `type-aware`, or `project-context`
4. Register in `packages/rules/src/registry/register-all.ts`
5. Add to the appropriate preset in `packages/rules/src/presets/`

Rules receive pre-filtered nodes in `handle(node, context)`. They must be stateless, allocation-minimal, and O(1) per node.

## Key Conventions

- Use typed `Result<T>` objects for errors, not thrown exceptions (except unrecoverable startup failures).
- Progress/debug output → `stderr`. Machine-readable output (JSON, SARIF) → `stdout`.
- Cross-package imports always use package names (`@ngcompass/common`), never relative paths across package boundaries.
- TypeScript strict mode, `module: "Node16"`, ES2022 target. Include file extensions in ESM imports.
- Do not lower coverage thresholds in `vitest.config.ts` — they are intentionally minimal during beta.
- When changing `RuleResult` shape or planner task format, bump the relevant `CACHE_SCHEMA_VERSION` or `PLAN_CACHE_VERSION` constant in `@ngcompass/cache`.

## License

PolyForm Shield 1.0.0. Free for any use except building a competing product. See `LICENSE`.

---

# Coding Standards

These standards exist because this is a performance-sensitive static analyzer. Violating them creates measurable regressions, not stylistic noise.

## Type Discipline

- **Never use `any`.** Use `unknown` and narrow with type guards.
- **Never use `as` casts** except for `as const` or narrowing from `unknown` after a runtime check. Use type guards.
- **Prefer `interface` for object shapes**, `type` for unions/intersections/utilities.
- **Mark fields and arrays `readonly`** when not mutated after construction.
- **No `Record<string, any>` or `object`** as a parameter type — define a real interface.
- **Explicit return types on every exported function.**
- **No optional parameters with side-effect defaults** — pass values explicitly.

## Error Handling

- **Domain errors return `Result<T, E>`.** Don't throw for expected failures.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RoadmapDevelop/ngcompass](https://github.com/RoadmapDevelop/ngcompass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
