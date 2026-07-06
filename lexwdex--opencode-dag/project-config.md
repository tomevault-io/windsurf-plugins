---
trigger: always_on
description: - To regenerate the JavaScript SDK, run `./packages/sdk/js/script/build.ts`.
---

- To regenerate the JavaScript SDK, run `./packages/sdk/js/script/build.ts`.
- The default branch in this repo is `main`.

## Git Workflow (铁律)

```
feat/**, fix/** ──PR(Typecheck 门禁)──▶ dev ──push 触发全量测试──▶
    dev ──手动 release-fork──▶ prerelease 测试版
    dev ──PR(全量测试门禁)──▶ main ──手动 release-fork──▶ 正式版
```

**分层门禁**：`dev` 是快速集成层（仅 Typecheck），`main` 是正式质量门禁（Typecheck + 全量 Unit Tests + E2E）。所有改动通过 PR 流转，禁止直推 `main` 和 `dev`（由 GitHub Rulesets 强制）。

| Branch | 直推 | PR 门禁 | CI 触发 | Purpose |
|--------|------|---------|---------|---------|
| `{type}/**` | ✅ 允许 | — | ❌ 不跑 | 开发分支，频繁变更 |
| `dev` | ❌ 禁止 | PR 必须通过 **Typecheck** | ✅ push 触发 Typecheck + 全量测试 | 快速集成层 |
| `main` | ❌ 禁止 | PR 必须通过 **Typecheck + Unit Tests + E2E (linux + windows)** | ✅ push 触发全量 | 正式质量门禁 + 发版 |

**流程**：
1. 从 `main` 切出 `feat/**` 或 `fix/**` 分支开发
2. PR → `dev`（Typecheck 门禁，快速合并）
3. push 到 `dev` 自动触发全量测试验证
4. 从 `dev` 手动 `release-fork` → 产出 **prerelease** 测试版
5. PR `dev` → `main`（全量测试门禁：Typecheck + Unit Tests + E2E）
6. 合并到 `main` 后手动 `release-fork` → 产出**正式版**

**Rulesets（GitHub Settings → Rules → Rulesets）**：
- `protect-main`：禁止直推/删除/force-push；PR 需通过 4 项检查（Typecheck、Unit Tests (linux)、E2E Tests (linux)、E2E Tests (windows)）
- `protect-dev`：禁止直推/删除/force-push；PR 需通过 Typecheck
- `branch-naming`：只允许创建 `feat/**`、`fix/**`、`chore/**`、`docs/**`、`refactor/**`、`test/**`、`release/**`、`hotfix/**` 前缀的新分支

**CI 配置**：
- `ci-typecheck.yml`：push 到 `main`/`dev` + PR → `main`/`dev` 时触发（快速门禁）
- `ci-test.yml`：push 到 `main`/`dev` + PR → `main` 时触发全量测试（`cancel-in-progress: false` 保证跑完）
- `release-fork.yml`：手动触发；从 `dev` 发布自动标记 `--prerelease`，从 `main` 发布正式版

## Branch Names

Format: `{type}/{short-name}` where `type` is one of: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `release`, `hotfix`. The short name uses hyphens, at most three words. Enforced by GitHub Ruleset `branch-naming`.

Examples: `feat/session-recovery`, `fix/scroll-state`, `docs/branch-naming`, `refactor/dag-spawn`, `test/auth-flow`, `chore/regenerate-sdk`, `release/v1.18`, `hotfix/critical-patch`.

## Commits and PR Titles

Use conventional commit-style messages and PR titles: `type(scope): summary`.

Valid types are `feat`, `fix`, `docs`, `chore`, `refactor`, and `test`. Scopes are optional; use the affected package or area when helpful, e.g. `core`, `opencode`, `tui`, `app`, `desktop`, `sdk`, or `plugin`.

Examples: `fix(tui): simplify thinking toggle styling`, `docs: update contributing guide`, `chore(sdk): regenerate types`.

## Style Guide

### General Principles

- Keep things in one function unless composable or reusable
- Do not extract single-use helpers preemptively. Inline the logic at the call site unless the helper is reused, hides a genuinely complex boundary, or has a clear independent name that improves the caller.
- Avoid `try`/`catch` where possible
- Avoid using the `any` type
- Use Bun APIs when possible, like `Bun.file()`
- Rely on type inference when possible; avoid explicit type annotations or interfaces unless necessary for exports or clarity
- Prefer functional array methods (flatMap, filter, map) over for loops; use type guards on filter to maintain type inference downstream
- In `src/config`, follow the existing self-export pattern at the top of the file (for example `export * as ConfigAgent from "./agent"`) when adding a new config module.
- In Effect generators, bind services to named variables before calling methods. Do not use nested service yields such as `yield* (yield* Foo.Service).bar()`.

Reduce total variable count by inlining when a value is only used once.

```ts
// Good
const journal = await Bun.file(path.join(dir, "journal.json")).json()

// Bad
const journalPath = path.join(dir, "journal.json")
const journal = await Bun.file(journalPath).json()
```

### Destructuring

Avoid unnecessary destructuring. Use dot notation to preserve context.

```ts
// Good
obj.a
obj.b

// Bad
const { a, b } = obj
```

### Imports

- Never alias imports. Do not use `import { foo as bar } from "..."` or renamed imports like `resolve as pathResolve`.
- Never use star imports. Do not use `import * as Foo from "..."` or `import type * as Foo from "..."`.
- If a namespace-style value is needed, import the module's own exported namespace by name, for example `import { Project } from "@opencode-ai/core/project"`, then reference `Project.ID`.
- Prefer dynamic imports for heavy modules that are only needed in selected code paths, especially in startup-sensitive entrypoints. Destructure dynamic import bindings near the top of the narrowest scope that needs them so they read like normal imports. Avoid inline chains such as `await import("./module").then((mod) => mod.value())` or `(await import("./module")).value()`. Keep branch-specific imports inside the branch that needs them to preserve lazy loading.

### Variables

Prefer `const` over `let`. Use ternaries or early returns instead of reassignment.

```ts
// Good
const foo = condition ? 1 : 2

// Bad
let foo
if (condition) foo = 1
else foo = 2
```

### Control Flow

Avoid `else` statements. Prefer early returns.

```ts
// Good
function foo() {
  if (condition) return 1
  return 2
}

// Bad
function foo() {
  if (condition) return 1
  else return 2
}
```

### Complex Logic


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeXwDeX/OpenCode-DAG](https://github.com/LeXwDeX/OpenCode-DAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
