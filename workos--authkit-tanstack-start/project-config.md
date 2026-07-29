---
trigger: always_on
description: Guidance for Claude Code working in this repository.
---

# CLAUDE.md

Guidance for Claude Code working in this repository.

## CRITICAL: File Organization and Import Boundaries

**DO NOT reorganize server-side code that works.** File structure matters for bundling.

### Lessons Learned

**Broken commits:**

- `5bd4367` — Reorganized `src/server/` → `src/auth/`, `src/core/`, `src/middleware/`
- `8058974` — Added `'use server'` directives

**Why it broke:**

1. Moving files changed import paths.
2. TanStack Start's bundler includes all imported modules at evaluation time.
3. When `authkit` was imported (even transitively), it immediately evaluated `@workos/authkit-session` and `iron-session`.
4. Server deps leaked into the client bundle → runtime errors.

**Why the original `src/server/` structure works:**

- Clear boundaries for the bundler to tree-shake on.
- The directory convention signals server-only intent to Vite.

**About `'use server'`:**

- NOT documented in TanStack Start.
- But works in practice when files are in `src/server/` (Vite bundler picks it up).
- Removing it causes crypto / iron-session to leak into client bundles.

**Lesson:** leave `src/server/` alone.

## CRITICAL: Lazy handler bodies (`actions.ts`, `server-functions.ts`)

`src/server/actions.ts` and `src/server/server-functions.ts` are reachable from `src/client/**` via type-only / RPC edges. Static value imports of server-only modules from these two files re-open the issue [#72](https://github.com/workos/authkit-tanstack-start/issues/72) class of leaks (e.g. `eventemitter3` SyntaxError in Vite dev once `@workos-inc/node` ships an awkward CJS dep).

**The pattern.** Each `createServerFn(...).handler(...)` body MUST be a thin shell that dynamically imports its real logic from the sibling bodies file:

```ts
// src/server/actions.ts
export const getAuthAction = createServerFn({ method: 'GET' }).handler(
  async (): Promise<ClientUserInfo | NoUserInfo> => {
    const { getAuthBody } = await import('./action-bodies.js');
    return getAuthBody();
  },
);
```

The actual logic lives in `src/server/action-bodies.ts` / `src/server/server-fn-bodies.ts`, which CAN statically import server-only modules.

**Why it works.** TanStack Start's compiler (`/\.[cm]?[tj]sx?($|\?)/`, no `node_modules` exclusion) transforms our installed `dist/server/*.js` in the consumer's client bundle, replacing each `.handler(fn)` with `createClientRpc(id)`. The shell's dynamic import then has no value reference in the client graph and is dead-code-eliminated. The bodies file is never reached from the client.

**Load-bearing assumption.** The TanStack compiler must keep transforming our installed dist. If a future Vite or TanStack release excludes `node_modules` from the transform pipeline, this approach degrades silently — the leak fires the first time client middleware invokes a handler. Re-verify by inspecting the served module after upgrades:

```bash
cd example && pnpm dev
# In another terminal:
curl -s 'http://localhost:3000/@fs/<absolute>/dist/server/actions.js' | grep -c 'createClientRpc'
# Expect non-zero. If zero, the compiler is no longer transforming the SDK's dist —
# stop and pivot to HTTP-RPC.
```

**Regression guard.** `.oxlintrc.json` configures `no-restricted-imports` (with `allowTypeImports: true`) on these two files, blocking static value imports of `./auth-helpers*`, `./authkit-loader*`, `./context*`, `./headers-bag*`, `./action-bodies*`, `./server-fn-bodies*`, `@workos/authkit-session`, and `@workos-inc/node`. Do NOT bypass the rule. If you need a new server module, add it to the bodies file and dynamic-import from the shell.

**Bundle check.** `pnpm run build:check` runs `scripts/check-bundle-leak.sh` against the example's built client bundle, looking for fingerprints (`@workos-inc/node`, `iron-session`, `iron-webcrypto`, `FeatureFlagsRuntimeClient`, `The listener must be a function`, `ERR_JWT_CLAIM_VALIDATION_FAILED`). Run after any change touching the `actions.ts` / `server-functions.ts` boundary or after upgrading `@workos/authkit-session`.

## CRITICAL: Server Function Execution Context

`createServerFn` creates automatic RPC boundaries — no directive needed.

Server functions can ONLY be called from server contexts:

| Context                 | Runs On                   | Can call server functions?  |
| ----------------------- | ------------------------- | --------------------------- |
| `loader`                | Server (SSR), then cached | Yes                         |
| `beforeLoad`            | Server AND client         | No                          |
| Server function handler | Server only               | Yes                         |
| Component render        | Server AND client         | No (use `useServerFn` hook) |
| Route server handlers   | Server only               | Yes                         |

### Correct: call from loader

```typescript
export const Route = createRootRoute({
  loader: async () => {
    const { user } = await getAuth();
    const url = await getSignInUrl({});
    return { user, url };
  },
});
```

### Wrong: call from beforeLoad

```typescript
export const Route = createRootRoute({
  beforeLoad: async () => {
    // beforeLoad runs on BOTH server and client (during hydration).
    const { user } = await getAuth(); // Throws: "can only be called on the server"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [workos/authkit-tanstack-start](https://github.com/workos/authkit-tanstack-start) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
