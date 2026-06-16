---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

AlloCat — minimalist personal-finance PWA, also shipped as a native Android app (Capacitor) for SMS-based transaction tracking. Next.js 16 (App Router) + React 19 + Supabase + Dexie (IndexedDB) + TanStack Query 5 + Tailwind v4. Currency is multi-currency via `lib/number-format.ts` + `lib/currency/catalog` (INR is the default for legacy callers); do **not** assume hardcoded `en-IN`.

`package.json` declares `name: "AlloCat-web"` despite the directory name.

## Commands

```bash
npm run dev         # next dev
npm run build       # next build && serwist build (service worker bundling)
npm run start       # next start
npm run lint        # eslint (flat config in eslint.config.mjs)
npm run test        # vitest run (one-shot)
npm run test:watch  # vitest (watch)

npx vitest run lib/sms/match.test.ts        # single file
npx vitest run -t "matches exact rule"      # single test by name
```

Test files live next to source (`*.test.ts`, e.g. `lib/ai/parseSmsTransaction.test.ts`, `lib/sms/match.test.ts`). No typecheck script — run `npx tsc --noEmit` if needed.

Use **pnpm** (per memory: `npm install` fails with ERESOLVE). Both `package-lock.json` and `pnpm-lock.yaml` are checked in.

### Android (Capacitor)

```bash
npx cap sync android                                  # copy web config + plugins into android/
CAP_SERVER_URL=http://192.168.1.20:3000 npx cap sync # point the shell at a LAN dev server
```

Requires Android Studio JBR (JDK 21). Open `android/` in Android Studio to build/run the APK.

## Required env (`.env.local`)

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_DEFAULT_KEY=
OPENROUTER_API_KEY=     # used by app/api/ai/chat
```

## Architecture

### Offline-first sync (the central pattern — read this before touching data flow)

Every page reads from IndexedDB first; the network is a fallback and a background reconciler. Three layers cooperate:

1. **IDB cache** — `lib/db/AllocatDB.ts` defines a Dexie schema mirroring the Supabase tables (`profiles`, `budgets`, `categories`, `budget_items`, `goals`, `assets`, `asset_categories`, `asset_value_history`, `debts`, `reports`, `net_worth_snapshots`, `activity_logs`, `merchant_rules`, `sms_transactions`) plus three sync infra tables: `sync_queue`, `id_map`, `sync_meta`. Currently at schema version 10. The DB is a browser-only singleton via `getDB()` — calling it on the server throws. Schema version bumps live in `AllocatDB.ts` constructor; add a new `.version(N).stores({...})` block, never mutate prior versions.

2. **Hydration + prefetch** — on mount, `SyncProvider` (`lib/providers/SyncProvider.tsx`) calls `hydrateAllTables()` (`lib/db/hydrate.ts`) which bulk-pulls every table for the current user into IDB. If `sync_meta.__userId__` differs from the active user, IDB is wiped first (multi-account device safety). After hydration, `prefetchAllQueries()` (`lib/db/prefetch.ts`) warms the React Query cache from IDB so first navigation has no skeletons. Use `qc.fetchQuery` (not `prefetchQuery`) when adding new prefetched keys — staleTime semantics would otherwise serve stale entries across reloads.

3. **Mutation queue** — mutations write to IDB optimistically (with a `temp_<uuid>` id for INSERTs), then `useEnqueue()` appends a `SyncQueueItem` to `sync_queue`. `SyncEngine` (`lib/sync/SyncEngine.ts`) drains the queue: each `(table, operation)` pair maps to a server action via the `dispatch` table — when adding new tables/operations, you must register a dispatcher entry there or the item will permanently fail. Failed items retry up to `MAX_RETRIES = 3` with backoff; permanent failures invoke `onRollback` (which invalidates relevant React Query keys). `temp_` ids inside payloads are rewritten to real ids via `id_map` before the action fires — use `extractTempIds` patterns when designing new payloads.

Cross-cutting rules:
- Server actions live in `lib/actions/<domain>.ts` and are the *only* path that talks to Supabase from the client side. They are also called directly during initial fetch (IDB miss) and via SyncEngine on flush.
- Read hooks live in `lib/hooks/use<Domain>.ts`. The pattern is: `getXFromIDB()` first; on miss, fall back to the server action. Each hook exports its query key constant (e.g. `DASHBOARD_KEY`, `budgetKey(month, year)`) — reuse these for invalidation.
- Mutation hooks must: (1) write to IDB optimistically, (2) `enqueue` the operation, (3) invalidate matching query keys in `onSuccess`.

### Routing

- `app/(app)/*` — protected app shell (dashboard, budget, debt, goals, net-worth, profile, activity, **sms**). Layout wraps in `TourProvider` → `SyncProvider`, with mobile-first 480px frame and `md:` desktop layout.
- `app/auth/*` — login / signup / oauth callback.
- `app/onboarding/page.tsx` — post-signup flow.
- `app/share-target/` — PWA Web Share Target landing (manifest `share_target` posts here); shared text is parsed by `lib/ai/parseSpend.ts`.
- `app/api/ai/chat/route.ts` — streaming AI chat. Hard off-topic regex guard runs *before* the model call; topic detection in `lib/ai-utils.ts` decides which slice of `buildFinancialContext` to attach.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BuiltByOctane/allocat](https://github.com/BuiltByOctane/allocat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
