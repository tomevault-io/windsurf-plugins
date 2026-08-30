---
trigger: always_on
description: **Analysis Date:** 2026-06-26
---

# Coding Conventions

**Analysis Date:** 2026-06-26

## Naming Patterns

**Files:**
- kebab-case for all TypeScript source files: `provider-launch.ts`, `mcp-runtime.ts`, `command-resolution.ts`, `claude-print.ts`
- PascalCase for Vue single-file components: `ChatInput.vue`, `MessageBubble.vue`, `ApprovalDialog.vue`
- Test files mirror their source: `provider-launch.test.ts` sits next to `provider-launch.ts`

**Functions:**
- camelCase throughout: `buildClaudeLaunchArgs`, `escapeSqliteLiteral`, `collectVersionManagerCommandCandidates`
- Verb-noun prefix convention: `build*`, `create*`, `get*`, `normalize*`, `resolve*`, `render*`
- Boolean predicates: `is*` or `has*` — `hasCaidoContextChanged`, `isFileNotFound`

**Variables:**
- camelCase: `mcpTempDir`, `sessionCaidoToken`, `currentSettings`
- Module-level mutable state uses `let`; stable references use `const`
- Module-level constants: SCREAMING_SNAKE_CASE — `CLAUDE_DISALLOWED_TOOLS`, `NODE_EXECUTABLE_ERROR`, `DEFAULT_CHAT_TITLE`

**Types and Interfaces:**
- PascalCase: `ClaudeLaunchInput`, `McpPermissionGroups`, `PersistenceDbHandle`
- Discriminated union members use string literal `kind` field: `{ kind: "Ok" }` / `{ kind: "Error" }`
- `type` preferred over `interface` for object shapes; `interface` appears only in `persistence.ts` for duck-typed handle

## Code Style

**Formatting:**
- Prettier 3.8.1 (`prettier` in root devDependencies)
- Run: `pnpm format` — formats `packages/**/src/**/*.{vue,ts,js,json}` plus root-level `*.{ts,mjs}` (`caido.config.ts`, `vitest.config.ts`, `vitest.setup.ts`, `eslint.config.mjs`)
- No `.prettierrc` committed — default Prettier settings apply (2-space indent, double quotes)

**Linting:**
- ESLint 10.8.1 flat config at `eslint.config.mjs` (root; `.mjs` because the root package is CommonJS-typed) — covers TypeScript, Vue SFCs, `.mjs`, and test files
- `pnpm lint` runs `eslint . --max-warnings 0`; it never auto-fixes, so CI cannot silently rewrite the checkout and report success. `pnpm lint:fix` is the local auto-fix entry point
- `pnpm typecheck` runs `tsc --noEmit` (backend) and `vue-tsc --noEmit` (frontend)

**Section headers in large files:**
- `index.ts` uses ASCII box headers to delimit logical sections:
  ```typescript
  // ── Types (inline to avoid Zod which crashes QuickJS) ──────────────
  // ── Persistence ─────────────────────────────────────────────────────
  // ── API: Settings ───────────────────────────────────────────────────
  ```
- Use this style when adding new top-level sections to `packages/backend/src/index.ts`

## Backend Constraints (QuickJS Runtime)

The Caido backend runs in a constrained **QuickJS** environment. This imposes hard rules:

**No Zod:** Zod crashes QuickJS. All runtime validation is done with hand-written type guards. See the comment at `packages/backend/src/index.ts:68`:
```typescript
// ── Types (inline to avoid Zod which crashes QuickJS) ──────────────
type CaidoValidationResult = ...
```

**No `crypto` module:** UUID generation uses a hand-rolled implementation in `packages/backend/src/index.ts:829`:
```typescript
/** Generate UUID v4 without crypto module */
function genUUID(): string {
  const hex = "0123456789abcdef";
  let uuid = "";
  for (let i = 0; i < 36; i++) {
    if (i === 8 || i === 13 || i === 18 || i === 23) {
      uuid += "-";
    } else if (i === 14) {
      uuid += "4";
    } else if (i === 19) {
      uuid += hex[(Math.random() * 4 | 8)];
    } else {
      uuid += hex[(Math.random() * 16 | 0)];
    }
  }
  return uuid;
}
```
Do not replace this with `crypto.randomUUID()` — it is unavailable in QuickJS.

**No parameterized SQLite queries:** The Caido SQLite binding does not expose parameterized query support. SQL strings are built manually with `escapeSqliteLiteral` in `packages/backend/src/index.ts:193`:
```typescript
function escapeSqliteLiteral(value: string): string {
  return value.replace(/'/g, "''");
}
```
The inputs are constrained at call sites: `key` is always a hardcoded string (`"settings"` or `"chats"`), and `value` is the JSON payload. New SQL calls must follow the same constraint and document it.

**Available standard modules in backend:** `fs/promises`, `child_process`, `buffer`, `path` — all used in `packages/backend/src/index.ts`. No third-party runtime dependencies.

## The Result Pattern

The `Result<T>` discriminated union is defined in `packages/shared/src/result.ts` and used as the return type for every backend RPC handler:

```typescript
export type Result<TOk = void, TErr = string> =
  | { kind: "Ok"; value: TOk }
  | { kind: "Error"; error: TErr };

export const Result = {
  ok: <TOk>(value: TOk): Result<TOk> => ({ kind: "Ok", value }),
  err: <TOk = never, TErr = string>(error: TErr): Result<TOk, TErr> => ({ kind: "Error", error }),
  isOk: ...,
  isErr: ...,
};
```

The backend `index.ts` re-declares a local version (same shape, different namespace) to avoid importing shared in the QuickJS context:
```typescript
type Result<T> = { kind: "Ok"; value: T } | { kind: "Error"; error: string };
function ok<T>(value: T): Result<T> { return { kind: "Ok", value }; }
function err<T>(error: string): Result<T> { return { kind: "Error", error }; }
```

Frontend stores call `result.kind === "Ok"` / `result.kind === "Error"` directly (no shared helpers needed).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [six2dez/drift](https://github.com/six2dez/drift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
