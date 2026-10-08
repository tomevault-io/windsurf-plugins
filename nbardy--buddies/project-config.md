---
trigger: always_on
description: Index for agents. Hard rules inline; read the linked doc BEFORE working in
---

# AGENTS.md — Agentic Coding Guide

Index for agents. Hard rules inline; read the linked doc BEFORE working in
that area.

## Code tree map

```
shared/src/index.ts                → shared exports + WS Zod schemas/types (protocol v3)
shared/src/conversation.ts         → list row / detail / message page / RowPatch (the wire model)
shared/src/conversation-config.ts   → canonical selection intent, patches, resolution
shared/src/provider-catalog.ts      → provider identities + catalog schemas
server/src/server.ts               → application composition and startup
server/src/conversations/runtime.ts → Conversation: record + queue + runner + kind policy
server/src/turns/*                 → TurnQueue, TurnRunner (event fold), TurnWatchdog, TurnPolicy
server/src/conversations/config-{service,records}.ts → durable configuration and revisions
                                     (records: Rust ConversationRecords store, own SQLite file)
server/src/transport/conversation-websocket.ts → WS command routing
server/src/observability/error-journal.ts → durable grouped server/client failures
server/src/ingest/*                → transcripts + list from the Rust ingest addon; runtimes
server/src/auth/*                  → shared-secret gate (policy/gate/express)
server/src/providers/*             → provider registry + catalog service (models from the generated catalog)
server/src/buddies/*               → Buddy server over the crate: grants, mcp (one HTTP
                                     endpoint, 13 tools), runner, channels, routes,
                                     briefing, memory-review, policy-port (T08 seam)
crates/unleashd-buddies/           → Buddies core (Rust, napi-rs addon): schema,
                                     authorize, posts/docs/tasks/runs
crates/unleashd-ingest/            → transcript ingest + conversation records (Rust addon)
vendor/agent-cli-tool/             → GIT SUBMODULE: canonical request → argv →
                                     process → unified event stream. Thin wrapper;
                                     harness differences live at its edges only.
client/src/atoms/*                 → jotai atoms, derived views, WS actions
client/src/components/{Sidebar,Chat,ConversationConfigPicker}.tsx → main desktop UI
client/src/mobile/*                → mobile view tree (second shell, same core)
client/src/atoms/ui.ts             → device-local UI prefs + NEW-badge seen indexes
{server,client}/src/swarm/          → QUARANTINED swarm/oompa viewer; outside code imports
                                     only swarm/index.ts (guard: swarm-quarantine.test.ts)
```

## Hard rules (violations = rejected PR)

- Never `useAtomValue(rowsAtom)` in components — use `rowFamily(id)` (the row)
  and `listField(...)` (one field of the one-pass `listIndexAtom`). Bodies and
  detail live in `transcriptFamily(id)`, loaded on open (`useConversationBodies`).
  Streaming text goes to `streamFamily(id)`, never the transcript mid-stream.
  Socket/server state is `connectionAtom`; in-flight commands `commandsAtom`
  (`commandFor(id)`). The route owns the active conversation id.
- All hooks before any early `return`. New list views go in derived atoms, not
  component `useMemo`. Stable fallbacks are module constants.
- Read-only server data goes through `usePolledFetch(source, intervalMs)`,
  never a bare `fetch(...).then(setState)` inside `useEffect`. Results live in
  the keyed cache in `client/src/atoms/resources.ts`, NOT in component state,
  so a remount renders the cached value immediately and revalidates behind it —
  that is what stopped every mobile page visit from waiting on a round trip.
  The cache key is the identity of the DATA: a plain URL string, or
  `resource(key, load)` when one load fans out into several requests (thread
  `signal` into every inner `fetch`). A bare `(signal) => Promise<T>` fetcher is
  rejected by the type — with no key there is nothing to cache under, and it is
  the un-keyed form that made seven panels show the previous project's data
  after a fast switch until 2026-09-06 (SwarmDetail, SwarmAnalytics,
  UsagePanel). Keying supersedes the old abort-on-change guard: a late response
  lands on its own key, which whoever switched away is no longer reading, so
  never re-add a `data.id === currentId` check at a call site.
- `jotaiStore.set` only inside `client/src/atoms/`. The shells never import
  each other (mobile/ vs `components/*` outside `components/buddies/`);
  `views/` imports neither. Width `@media` lives only in shell CSS (G9); a
  view's device tweak is a colocated `[data-device="mobile"] .x` rule.
  Gates: `bash tools/check-client-invariants.sh`.
- One WS bridge (`App.tsx`), one `handleMessage` spine — never a second.
- Auth gate stays FIRST in the Express chain and the WS stays `noServer` +
  explicit `upgrade` handler. `new WebSocketServer({ server })` accepts every
  upgrade before app code runs and republishes the whole command channel.
  Same for the Vite dev server: Connect middleware never sees `upgrade`, so
  the HMR socket needs its own `prependListener('upgrade')` gate.
- Provider-bespoke values (effort levels etc.) pass through verbatim as
  `z.string()`; no shared enums, no value translation, server-side defaults.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nbardy/buddies](https://github.com/nbardy/buddies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
