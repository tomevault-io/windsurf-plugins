---
trigger: always_on
description: enables seamless tab switching without re-scanning. Do not move this to React
---

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# at.fund — Agent Guide

Quick-reference for AI agents working in this codebase. Full architecture docs
live in `docs/pipeline.md`; this file covers the guardrails and invariants you
must not break.

## What the app does

Helps ATProto users discover and fund the tools, feeds, and labelers they rely
on. Users sign in with Bluesky OAuth, the app scans their account relationships,
and renders funding cards for each discovered service.

## Architecture overview

```
Client (React 19)          Server (Next.js app router)
─────────────────          ────────────────────────────
Profile page               GET /api/stack/[handle]/stream (NDJSON)
  ProfileClient               resolveEntry() × N endorsed URIs
  ├── ProfileCard              resolveDependencies() (BFS)
  ├── StackStream          GET /api/lexicons/stream (NDJSON)
  │   └── StackEntriesList     scanStreaming() — 6-phase pipeline
  └── SetupClient (edit)

Give page
  GiveClient               GET /api/lexicons/stream (NDJSON)
  ├── StackEntriesList         scanStreaming() orchestrator
  └── HandleAutocomplete       6-phase pipeline:
                                 1. Gather accounts (follows, repo NSIDs, feeds, labelers)
                                 2. Collect network endorsements (single-pass over follows)
                                 3. Discover ecosystem entries (endorsement map lookup)
                                 4. Enrich (fund.at records, manual catalog, profiles)
                                 5. Attach capabilities (feeds, labelers, PDS)
                                 6. Resolve dependencies (breadth-first)
```

All phases receive a `ScanContext` — the single network orchestrator that owns
prefetch, caching, and concurrency. See `lib/scan-context.ts`.

## Critical invariants — do not break these

### ScanContext threading
Every pipeline phase and standalone resolver receives `ScanContext`. Never create
a second context within an existing scan. Never fetch fund.at records directly
when a context is available — always check `ctx.fundAtPrefetch` first.

### Module-level scan cache (`useScanStream`)
`hooks/useScanStream.ts` maintains a module-level `_scanCache` that survives
client-side navigation but clears on hard refresh. This is intentional — it
enables seamless tab switching without re-scanning. Do not move this to React
state or context (it must survive component unmounts). Currently only consumed
by `GiveClient.tsx`; do not add a second consumer without a guard.

### Streaming NDJSON contract
`/api/lexicons/stream` emits newline-delimited JSON events. The event types and
their order are documented in `docs/pipeline.md` (Event types section). Adding
new event types is fine; changing existing event shapes is a breaking change to
the client parser in `useScanStream`.

### Canonical types (DID-first)
- `Identity` — resolved presentation. `did` is required; `uri` always equals `did`. Handles and hostnames are display metadata only.
- `Funding` — how to contribute (source, contributeUrl, dependencies, channels, plans)
- `StewardEntry` — `Identity & Funding & { tags, capabilities }`
- `FundingChannel` / `FundingPlan` — payment channel and tier types from `lib/funding-manifest.ts`

All identity resolution goes through `buildIdentity()` in `steward-model.ts`.
All funding resolution goes through `resolveFunding()` / `resolveFundingForDep()`
in `funding.ts`. Do not create ad-hoc resolution logic. The manual catalog is
indexed by hostname (filename) with a DID reverse index — lookups by DID work
natively via `lookupManualStewardRecord()`.

### Lexicon NSIDs
New grouped NSIDs: `fund.at.actor.declaration`, `fund.at.funding.contribute`,
`fund.at.funding.channel`, `fund.at.funding.plan`, `fund.at.graph.dependency`,
`fund.at.graph.endorse`. Legacy flat NSIDs (`fund.at.contribute`,
`fund.at.dependency`, `fund.at.endorse`) are kept for migration reads only.
Always use the exported constants from `lib/fund-at-records.ts` — never
hardcode NSID strings. Read paths try new NSIDs first, fall back to legacy.
Writes always use new NSIDs.

### Auth flow
OAuth with ATProto DPoP. Session stored in an httpOnly DID cookie + Redis KV
(with in-memory fallback for local dev). Session validation happens on route
change, tab focus, and 401 response. The DPoP fetch patch in `lib/auth/client.ts`
works around a Next.js ReadableStream issue — do not modify without testing the
full OAuth flow.

**Cookie-based identity:** The `did` and `handle` cookies are both set at OAuth
callback time. The root layout reads them synchronously — no `getSession()` call,
no network I/O. This means every page render is instant regardless of session
state. `getSession()` is only called when the actual OAuth session object is
needed (e.g. `fetchOwnFundAtRecords` in the profile page's owner mode).

**Profile page session check:** The `/<identifier>` route uses `getSession()` to

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andyschwab/at.fund](https://github.com/andyschwab/at.fund) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
