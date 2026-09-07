---
trigger: always_on
description: > **Scope:** ONLY rules, invariants, load-bearing pairs and traps invisible at the call site. Not a tour: open the file. Where the code already comments the trap, one line here. When a change makes a rule wrong, fix it in the same commit.
---

> **Scope:** ONLY rules, invariants, load-bearing pairs and traps invisible at the call site. Not a tour: open the file. Where the code already comments the trap, one line here. When a change makes a rule wrong, fix it in the same commit.

Next.js 16 + Elysia BFF over our `new-api` fork. Local-first: one SQLocal/OPFS DB per device in the browser is the SOLE copy of chat/RP state; Turso holds only the model-tester rankings. "Sync" means atom<->local DB. Never add a server-side mirror, a `/sync` route, a server context cache, or a retry that expects the server to hold context.

## Dev

- NEVER start, restart or kill the dev server (`bun dev:log`, logs in `/tmp/next.log`). Read the log.
- NEVER `bun run build` or `rm -rf .next` while dev may run: shared turbopack cache, corrupts or kills the live server. Verify builds in a worktree:

```bash
git worktree add /tmp/uno-build HEAD && cd /tmp/uno-build && bun install && bun run build
git worktree remove --force /tmp/uno-build
```

- Local SSR hits `api.unorouter.com` without a session; `EDGE_DEV_TOKEN` in `.env` (value in `infra/infra/cloudflare/unorouter.com/rules*.sops.yaml`) exempts it. Symptom when stale: `Just a moment...` 403 in the log.
- `src/openapi.ts` is Orval output, never edit; `bun openapi`.

## Rules

- Translation keys UPPER_SNAKE nested, `msg()` outside React. `de.d.json.ts` is generated, not a locale.
- Dates via the `dayjs` singleton (`src/lib/utils/format/date.ts`), never raw `Date`.
- Enum-likes are TypeBox `t.Union([t.Literal()])` + derived type. No new TS `enum`; `src/lib/types/enums.ts` is grandfathered.
- `as`: delete the cast and run `bun typecheck`; still green means it was dead.
- Never refactor `src/components/ui/` for conciseness (shadcn/assistant-ui primitives).
- Local DB mutation, then `invalidateQueries`. No `setQueryData` patches, no optimistic rollback. React Query defaults `staleTime: Infinity`, no focus refetch, so nothing invalidates itself.
- ONE local database per device: `getLocalDb()` takes no argument. Only the model-tester tables carry `userId`, from `useAuthUserId()`/`authUserId()`. No client user-id atom, no per-user OPFS path.
- Pass upstream shapes through; when the shape is wrong, fix it in new-api (ours), never remap, filter, sort, count or cache in the BFF. Target route body: `async () => getSomething()`. BFF keeps auth, cookies, composing several upstream calls and unorouter-only view state. `handleElysia` unwraps `{success, data}` and DROPS siblings, so our own DTOs never use that envelope. Pass-through routes `return unwrap(res)`; local-logic routes (`ai/chat`, `ops/health`) `return { success, data }`; never mix in one route. No try/catch in handlers.
- Dependency direction one way: `route.ts` -> `*.service.ts` -> data. `src/lib/ai/chat/` is PURE of secrets and data sources; everything injected through `AssemblerDeps` (`pipeline/deps.ts`). `providers/` and `agents/` are leaf modules.
- Any remote fetch goes through `src/lib/config/safe-fetch.ts` (SSRF policy). Analytics events only via `src/lib/analytics.ts`; posthog-js is never statically imported.
- nuqs owns URL filter state; never `useSearchParams` + `router.replace`. `/models` bridges nuqs to jotai at the store level (cookie hydration lands after first-commit subscribers).
- jotai: `chatStore.set(atom, fn)` runs fn as an updater. `set(atom, () => fn)`.

## Rendering and caching

- No COOP/COEP anywhere (opfs-sahpool needs none; COEP forced full reloads and broke embeds). Badge route sets CORP on its own responses.
- `cacheComponents` OFF, no `use cache`: it double-rendered every cookie-reading route. React `cache()` is per-request dedup; the only TTL caches are module-state (benchmarks permaslug 30d, runware catalog 30min). Anything feeding a module cache must NOT fetch with `customFetch` defaults: it attaches the caller's cookies, so a URL key serves one user's data to all.
- ZERO `<Suspense>` in `src/`. A suspending hook or `next/dynamic` chunk suspends to the ROOT; if that bites, one targeted boundary at the component, never a layout gate.
- `(sidebar)` layout IS the auth gate (awaits self lookup, `redirectToLogin()`); it stays a plain async layout, a Suspense wrapper commits 200 before the redirect.
- Next 16 does not fire `[locale]/not-found.tsx` for child `notFound()`; each route group needs its own.

## Client DB (SQLocal / opfs-sahpool)

Four guards in `src/lib/db/client/`, each commented at the site with its data-loss mode. Read before touching: orphaned-pool guard (`assertNotSilentlyEmptied`, non-recoverable ON PURPOSE, bytes stay for `sahpool/salvage.ts`), never-blind-wipe retries in `openMigratedSql`, multi-tab handover (`want` re-sent on an interval; hidden tab parks immediately because frozen owners never answer), Safari legacy copy (legacy file removed even when the copy fails, else it re-imports over newer data).

- Migrations: forward-only, NO transaction wrapper, three passes (replay, `reconcileSchema` which aborts rather than drop ROWS, `validateColumns`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unorouter/unorouter](https://github.com/unorouter/unorouter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
