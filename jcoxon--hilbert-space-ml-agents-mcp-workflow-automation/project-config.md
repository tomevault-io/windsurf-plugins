---
trigger: always_on
description: Open-source AI-first workflow automation platform. Self-hosted or cloud. 400+ pieces. MCP support.
---

# Activepieces

Open-source AI-first workflow automation platform. Self-hosted or cloud. 400+ pieces. MCP support.

## Architecture (Non-Obvious Rules)

- **Multi-tenant**: Platform → Projects → Users. ALL queries MUST filter by `projectId` or `platformId`.
- **Editions**: CE (`ce`), EE (`ee`), Cloud (`cloud`) via `AP_EDITION`. EE extends CE via `hooksFactory` — **never import `src/app/ee/` in CE code**.
- **Feature gating**: `platformMustHaveFeatureEnabled((p) => p.plan.myFlag)` on EE modules.
- **Entity registration**: New entities MUST be added to `getEntities()` in `database-connection.ts` — TypeORM does NOT auto-discover.
- **HTTP**: `POST` for all create/update mutations. `DELETE` for deletes. Never PUT/PATCH.
- **Security**: Every endpoint needs `securityAccess` config.
- **Side effects**: Separated into `*-side-effects.ts` files, called explicitly after mutations.
- **Multi-server**: Use `distributedLock`, BullMQ deduplication, or `FOR UPDATE SKIP LOCKED` for concurrent operations.
- **Managed PostgreSQL**: No custom extensions. Use `sanitizeObjectForPostgresql()` for external data.
- **Before modifying a module**: Read its `.agents/features/<name>.md` file for entities, services, and integration details.
- **Cross-cutting libraries live in `packages/core/*`**, ordered thin → thick: `core-utils`, `core-piece-types`, `core-formula`, `core-execution` (thin, bundleable, framework-agnostic) and `core/shared` (the one thick, app-level member — **keeps the name `@activepieces/shared`**, carries DB/EE/management schemas + heavy deps). Pieces and the engine may import the thin members but **never** `@activepieces/shared`; pieces get what they need via `@activepieces/pieces-framework`. See `.claude/rules/core-packages.md`.
| `.agents/features/*.md` | ~60 lines each | When Claude explores the feature | Entity schemas, services, data flows |
| `.claude/rules/` | 3-5 lines each | Every session | Critical safety checks (entity registration, data isolation, edition safety) |
| `.agents/skills/` | 30-65 lines each | When invoked | Step-by-step workflows (`/add-feature`, `/add-entity`, `/add-endpoint`) |
- **Exported types and constants must be placed at the end of the file**, after all logic (functions, hooks, components, classes, etc.). This keeps the logic front and centre when reading a file, and groups the public contract at a predictable location.

  ```ts
  // ✅ Correct
  function doSomething() { ... }

  export const MY_CONST = 'value';
  export type MyType = { ... };
  // ✅ Correct
  const businessService = () => { ... }

  export const MY_CONST = 'value';
  export type MyType = { ... };

  // ❌ Wrong — types/consts mixed in before logic
  export const MY_CONST = 'value';
  export type MyType = { ... };
  function doSomething() { ... }
  ```

## Coding Conventions

- **No `any` type** — Use proper type definitions or `unknown` with type guards
- **No type casting** — Do not use `as SomeType` to force types. If you encounter an unnecessary cast, remove it.
- **No deprecated APIs** — Before using any library method or export, check its JSDoc. If it carries a `@deprecated` tag, use the recommended replacement instead. Examples: prefer `z.enum` over `z.nativeEnum`.
- **Go-style error handling** — Use `tryCatch` / `tryCatchSync` from `@activepieces/shared`
- **Zod error messages must be i18n keys** — Every `.min()`, `.refine()`, `.superRefine()`, etc. that surfaces a user-facing message must pass a string that exists as a key in `packages/web/public/locales/en/translation.json`. For common messages (e.g. required fields) use the `formErrors` constant from `@activepieces/shared`. Add a new translation key if none fits; never use raw English sentences that are not in the translation file.
- **`@activepieces/shared` version bump** — Any change to `packages/core/shared` must be accompanied by a version bump in `packages/core/shared/package.json`: bump the **patch** version for non-breaking additions or fixes, bump the **minor** version for new exports or behaviour changes after you check if it has already been bumped in the current branch or not
- **Helper functions** — Define non-exported helpers outside of const declarations
- **Named parameters** — Always use a single destructured object parameter instead of positional arguments. This applies to every function with more than one parameter, regardless of type. It prevents mix-ups at the call site and makes future additions non-breaking.
- **Prefer immutable data flow** — Functions should produce data by returning it, not by mutating an array/object the caller passes in. If a helper accumulates results (logs, derived rows, computed bindings), it should build the collection locally and return it — not take a pre-allocated bag the caller will read after. Local mutation inside a function's own body is fine; mutation that crosses the function boundary is not. Build new collections with `.map` / `.filter` / `.reduce` / spread rather than in-place `push` / `splice` / property assignment when feasible.
- **File order**: Imports → Exported functions/constants → Helper functions → Types

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JCoxon/hilbert-space-ml-agents-mcp-workflow-automation](https://github.com/JCoxon/hilbert-space-ml-agents-mcp-workflow-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
