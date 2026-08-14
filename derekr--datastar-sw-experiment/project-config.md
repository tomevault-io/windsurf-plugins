---
trigger: always_on
description: Local-first kanban board where a service worker is the server. Datastar handles UI reactivity, Hono + JSX in the SW handles routing/rendering/persistence.
---

# CLAUDE.md

Local-first kanban board where a service worker is the server. Datastar handles UI reactivity, Hono + JSX in the SW handles routing/rendering/persistence.

## Commands

```bash
pnpm dev          # Start dev server on localhost:5173
pnpm build        # Production build to dist/
```

## Architecture

`sw.jsx` is the entire server (~2600 lines). It contains Hono routes, JSX components, CSS, event sourcing, and idb persistence. `eg-kanban.js` handles pointer drag and keyboard navigation. `index.html` bootstraps the SW.

### CQRS flow

1. Command routes (`POST`/`PUT`/`DELETE`) write events → apply to projection → return `204`
2. Bus event notifies active SSE streams
3. Each SSE handler reads full state from IndexedDB → pushes complete HTML morph
4. Datastar morphs the DOM via Idiomorph

### The Tao of Datastar

- **Server pushes fat morphs** — full board re-render on every mutation via SSE `datastar-patch-elements`
- **Fewer signals is better** — just enough to communicate intent to the server
- **Server tracks UI state** — action sheets, selection mode, editing state are all in the SW's in-memory `boardUIState` Map. Mutations push full board morphs with the right UI baked in.

### Why CQRS: what complexity disappears

**Commands don't decide what to return.** In traditional REST, every mutation handler must figure out what data the client needs to update the UI — the full resource? related resources? computed aggregates? This scatters read/formatting logic across every endpoint and causes bugs where the response is missing something the UI needs (move a card, column count doesn't update because the response only returned the card). Here, every command returns `204`. The SSE handler is the single place that reads the full projection and renders `<Board />`. One read path, one render path.

**No stale siblings.** In request-response, after a mutation you must figure out what *else* on the page is now stale — the `queryClient.invalidateQueries` problem. Did creating a card change the column count? The board header badge? You have to manually enumerate what to invalidate, and you inevitably miss things. With SSE push, the bus dispatches `board:<id>:changed`, the SSE handler reads the entire board projection and pushes a complete morph. Everything is consistent by construction because it's all rendered from the same read.

**Multi-client sync is free.** Two tabs with the same board open both have SSE streams subscribed to `board:<id>:changed`. One tab creates a card, `appendEvents` fires the bus event, both streams push. No polling, no separate WebSocket layer, no "refetch on focus" logic. The tab count feature is a concrete example — it falls out of the architecture rather than being bolted on.

**No optimistic update rollback.** Traditional SPAs optimistically update the UI, send the mutation, and roll back on failure. This causes bugs: partial rollbacks, race conditions between concurrent optimistic updates, flicker when the server response disagrees. With CQRS+SSE where the SW is in-process, the latency between command and morph is near-zero, so optimistic updates are unnecessary.

**Command handlers become pure validation + intent.** When a handler doesn't format a response, it focuses entirely on "is this valid?" and "what happened?" The delete column handler (`sw.jsx:552-562`) is: clear the action sheet if needed, create a `column.deleted` event, append it, return 204. No conditional logic about what to include in the response.

**UI state and data state use the same push mechanism.** Action sheets, selection mode, editing state are ephemeral UI concerns that don't warrant events, but `emitUI()` uses the same SSE push path as data mutations. The SSE handler reads `getUIState()` alongside `getBoard()` and renders both into `<Board />`. One state-update pipeline on the client, not two.

### What event sourcing adds (beyond CQRS)

CQRS benefits come from the command/query separation itself. Event sourcing adds:

- **Undo/redo** — `lib/undo.js` builds reverse events by snapshotting state before the forward events. Without event sourcing, undo requires storing before/after snapshots (expensive) or inverse-operation logic per mutation type (brittle).
- **Time travel** — `replayToPosition` replays events up to a target index to reconstruct past board states. Only possible with the full event history.
- **Audit trail** — Card detail view shows the event history for that card via `loadCardEvents`. Just a query over the event store.
- **Idempotent replay** — `appendEvents` deduplicates by event ID (`lib/events.js:175`), making future sync safe: receive the same event twice, it's a no-op.
- **Schema evolution without migration** — Upcasters (`lib/events.js:4-15`) transform old event formats on read. No data migration needed — add an upcaster and the projection rebuilds correctly.

The cost: the event store grows (mitigated by snapshots in `lib/init.js`), and event schemas are append-only.

### Single-flight mutations vs CQRS+SSE


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derekr/datastar-sw-experiment](https://github.com/derekr/datastar-sw-experiment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
