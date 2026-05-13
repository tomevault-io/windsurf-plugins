---
trigger: always_on
description: Sandchest is a Linux-only sandbox platform for AI agent code execution. Every sandbox is a Firecracker microVM with VM-grade isolation, sub-second fork capability, and a permanent session replay URL.
---

# Sandchest — Development Guide

## Project Overview

Sandchest is a Linux-only sandbox platform for AI agent code execution. Every sandbox is a Firecracker microVM with VM-grade isolation, sub-second fork capability, and a permanent session replay URL.

**Polyglot monorepo:**
- `apps/api` — Control plane HTTP API (EffectTS on Node.js)
- `apps/web` — Dashboard + replay page (Next.js App Router)
- `packages/sdk-ts` — TypeScript SDK (`@sandchest/sdk`)
- `packages/mcp` — MCP server (`@sandchest/mcp`)
- `packages/cli` — CLI tool (`@sandchest/cli`)
- `packages/contract` — Shared types + protobuf definitions
- `packages/db` — PlanetScale schema + migrations
- `packages/config` — Shared ESLint/tsconfig base
- `crates/sandchest-node` — Rust node daemon (bare-metal Firecracker management)
- `crates/sandchest-agent` — Rust guest agent (runs inside microVM)

**Data infrastructure:** PlanetScale MySQL (metadata), Redis (ephemeral state/leasing), Scaleway Object Storage (artifacts, event logs).

---

## Package Manager

**Always use `bun`.** Never use npm, yarn, or pnpm.

```sh
bun install          # install dependencies
bun add <pkg>        # add a dependency
bun add -d <pkg>     # add a dev dependency
bun run <script>     # run a package.json script
```

For workspace operations:
```sh
bun install          # installs all workspace packages from root
```

---

## Testing

**Test suite: `bun test`** (built-in Bun test runner — Jest-compatible API).

```sh
bun test                          # run all tests
bun test packages/contract        # run tests in a specific package
bun test --watch                  # watch mode
bun test path/to/file.test.ts     # run a specific file
```

### Testing principles

- **Test behavior, not implementation.** Tests should survive refactors without changing.
- **Unit test pure functions and utilities aggressively** — especially ID generation, encoding, type transforms.
- **Integration tests for HTTP handlers** — test the full request/response cycle with mocked DB/Redis.
- **Co-locate tests with source** — `src/foo.ts` + `src/foo.test.ts` in the same directory.
- **No test utility files that are themselves untested.** Test helpers must be simple enough to trust.
- One `describe` per module, one `test` per behavior. Use `it` for user-facing behavior descriptions.
- Use `expect.assertions(n)` in async tests that assert inside callbacks to prevent silent passes.

```ts
// Good — tests behavior
test('generateId produces sortable IDs for the same prefix', () => {
  const a = generateId('sb_')
  const b = generateId('sb_')
  expect(a < b).toBe(true)
})

// Bad — tests implementation
test('generateId calls crypto.randomBytes', () => { ... })
```

---

## TypeScript Conventions

### Strictness

All packages use strict TypeScript (`"strict": true`). No exceptions. No `any` without a comment explaining why.

```jsonc
// tsconfig base
{
  "strict": true,
  "noUnusedLocals": true,
  "noUnusedParameters": true,
  "exactOptionalPropertyTypes": true
}
```

### Imports

- **ESM-first** — use `.js` extensions in imports even for `.ts` source files (NodeNext resolution).
- Prefer `import type` for type-only imports.
- No barrel re-exports that create circular dependencies.

```ts
import type { SandboxStatus } from '@sandchest/contract'
import { generateId } from './id.js'
```

### Error handling

Errors are **structured and typed**. Never throw raw strings. Every public-facing error has a `code`, `message`, and `requestId`.

```ts
// In SDK/API boundary code
throw new SandboxNotRunningError({
  message: `Sandbox ${sandboxId} is not in running state (current: ${status})`,
  requestId,
})
```

EffectTS is used in `apps/api` — use `Effect.fail` with typed error types, not `throw`. Let the runtime handle unhandled failures at the edge.

### Naming

- Types and interfaces: `PascalCase`
- Functions and variables: `camelCase`
- Constants and env vars: `SCREAMING_SNAKE_CASE`
- Files: `kebab-case.ts`
- Prefixed IDs in API responses: `sb_`, `ex_`, `sess_`, `art_` — never raw UUIDs

---

## React / Frontend Code

`apps/web` uses Next.js 15 App Router with React 19. Follow these rules strictly when writing any React component.

### You Might Not Need an Effect

Before reaching for `useEffect`, ask: **why does this code need to run?**

| Reason | Solution |
|--------|----------|
| Derived from props/state | Calculate during render (no hook needed) |
| Expensive calculation | `useMemo` |
| User interaction happened | Event handler |
| External system sync | `useEffect` ✓ |

**Never use Effects for these patterns:**

```tsx
// ❌ Effect to derive state
useEffect(() => {
  setFullName(first + ' ' + last)
}, [first, last])

// ✅ Calculate during render
const fullName = first + ' ' + last

// ❌ Effect to reset state on prop change
useEffect(() => {
  setSelection(null)
}, [userId])

// ✅ Key the component to reset it entirely
<Profile userId={userId} key={userId} />

// ❌ Chained effects
useEffect(() => { if (card.gold) setGoldCount(c => c + 1) }, [card])
useEffect(() => { if (goldCount > 3) setRound(r => r + 1) }, [goldCount])

// ✅ All state in one event handler

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CapSoftware/Sandchest](https://github.com/CapSoftware/Sandchest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
