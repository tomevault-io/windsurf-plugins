---
trigger: always_on
description: Notes for AI agents working in this repo. Read this before editing anything.
---

# CLAUDE.md

Notes for AI agents working in this repo. Read this before editing anything.

## What this project is

An X-style social app whose ranking algorithm is a config file you can read and
edit. The product is the algorithm's legibility, not the social network.

**The one invariant: a person must be able to open `lib/algorithm/weights.ts`,
change a number, refresh, and watch the feed reorder.** Every decision here bends
to that. If a change makes the ranking faster but harder to follow, it is the
wrong change. Optimise for someone reading this cold.

Corollary: do not introduce a model, an embedding store, Redis, a queue, or a
separate backend. Everything runs inside Next.js plus Supabase, and demo mode
runs with neither.

## Commands

```bash
npm run dev      # localhost:3000. Works with an empty .env.local (demo mode).
npm run build    # production build. Typechecks as part of the build.
npm run lint     # eslint
npm run seed     # loads the demo world into a real Supabase project
npx tsc --noEmit --incremental false    # typecheck alone
```

There is no test suite. Verification is: typecheck, lint, build, then exercise
the routes by hand (see [Verifying a change](#verifying-a-change)).

## Architecture

Request flow for the feed, which is the only complicated path:

```
POST /api/feed  { weights, rules, seen, limit }
  -> lib/data/session.ts      who is asking, and which database
  -> lib/algorithm/home-mixer.ts
       -> candidate-pipeline.ts   ~200 eligible posts, in and out of network
       -> thunder.ts              in-memory post + count cache, 30s TTL
       -> phoenix.ts              13 action probabilities -> weighted sum
       -> back in home-mixer      visibility, dedupe, diversity, blending
  -> lib/algorithm/redact.ts   strip private counts
  -> FeedResponse
```

| Path | What lives there |
| --- | --- |
| `lib/algorithm/weights.ts` | **The point of the project.** 13 weights + 6 feed rules, one plain-English comment per line. |
| `lib/algorithm/phoenix.ts` | The ranker. 13 named predictor functions. Heavily commented on purpose. |
| `lib/algorithm/candidate-pipeline.ts` | Who competes. Never ranks the whole table. |
| `lib/algorithm/thunder.ts` | Cache. Has `invalidate(postId)`. |
| `lib/algorithm/home-mixer.ts` | Orchestrator plus the post-scoring rules. |
| `lib/algorithm/storage.ts` | Client-side tuned weights in localStorage. |
| `lib/algorithm/redact.ts` | Strips private negative counts before responses leave the server. |
| `lib/types.ts` | `ALL_ACTIONS` is the spine of the whole app. |
| `lib/data/session.ts` | `getSession()`. Every page and route starts here. |
| `lib/demo/` | The in-memory world used when Supabase is not configured. |
| `lib/supabase/` | Three clients: `client` (browser), `server` (cookies, RLS), `admin` (service role). |
| `supabase/migrations/` | Two SQL files, run in order in the Supabase SQL editor. |
| `scripts/seed.mjs` | Loads `lib/demo/dataset.ts` into a real project. |

## Landmines

These have all bitten someone already. Read them.

**1. Never `.upsert({ onConflict })` on `engagements`.**
The uniqueness rule is a *partial* index (`engagements_one_per_toggle_idx`).
Postgres cannot use a partial index as an `ON CONFLICT` arbiter unless the
statement repeats the index predicate, and PostgREST has no way to send one, so
an upsert fails with `42P10` on every call. Insert and swallow the duplicate:

```ts
const { error } = await supabase.from('engagements').insert(row);
if (error && error.code !== '23505') throw error;   // 23505 = unique_violation
```

**2. Server-only modules must never reach a `'use client'` file.**
`lib/supabase/admin.ts`, `lib/data/session.ts`, `lib/demo/store.ts`,
`lib/demo/db.ts`, `lib/demo/dataset.ts`. Importing the demo store into a client
component ships the entire dataset to the browser; importing admin leaks the
service role key. Check before you import.

**3. Anything leaving the server goes through `redact.ts`.**
The ranker reads global counts with the service role, bypassing RLS. But RLS
deliberately keeps `mute_author`, `block_author`, `report` and `not_interested`
private. `publicCounts()` and `redactFeedPosts()` strip them at the API
boundary. Remove that and you have routed around your own security policy.

**4. `notFound()` must run before any Suspense boundary.**
Once the shell has flushed, Next renders the 404 page with a **200** status.
Resolve the record in the page function, call `notFound()` there, and stream
only the secondary data. Both `app/post/[id]` and `app/profile/[username]` do
this deliberately. Do not move those lookups back inside `<Suspense>`.

**5. Never call `createClient()` unconditionally in a client component.**
`createBrowserClient` throws on empty env vars, and client components are
server-rendered, so it takes the whole page down in demo mode. Use the lazy
pattern from `Composer.tsx` and `ReplyComposer.tsx`:

```ts
const supabase = useMemo(() => (isDemo ? null : createClient()), [isDemo]);
```

**6. Foreign key constraint names are load-bearing.**
The code embeds relations by constraint name: `profiles!posts_author_id_fkey`
in `thunder.ts`, `posts!engagements_post_id_fkey` in `candidate-pipeline.ts`.
Rename a constraint in the migration and post authors stop loading.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ItsRealAJ/OpenSocial](https://github.com/ItsRealAJ/OpenSocial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
