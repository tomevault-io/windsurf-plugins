---
trigger: always_on
description: Dotflowy is an outliner: nested bullets, per-user sync, and a plugin-extended editor. It is a React SPA with no SSR, on Cloudflare Workers. Each user's outline lives in its own Durable Object.
---

# Dotflowy

Dotflowy is an outliner: nested bullets, per-user sync, and a plugin-extended editor. It is a React SPA with no SSR, on Cloudflare Workers. Each user's outline lives in its own Durable Object.

Keep this file brief. Put task-specific guidance behind a pointer.

## Gotchas

- **Structural vs field writes.** Tree-shape changes (insert, indent, outdent, move, reparent, remove, undo restore, daily get-or-create) go through `runStructural`: one batch, overlay held until echo. Field edits (`setText`, `setKind`, `setIsTask`, toggles) stay a direct PATCH. Wrap at the call sites, not inside `mutations.ts`.
- **A new `Node` field touches seven places:** `src/data/wire-schema.ts`, `src/data/schema.ts`, `makeNode()`, `withNodeDefaults` in `collection.ts`, a DO `ADD COLUMN` migration, `e2e/fixtures.ts`, and the R2 snapshot boundary in `worker/backup.ts`. Miss the fixtures and inbound-frame decode rejects every snapshot.
- **Add a new side-collection to `KV_COLLECTIONS` in `worker/index.ts`.** The e2e kv mock accepts any name.
- **Build nodes with `makeNode()`.** A schema default makes the field optional in the encoded type.
- **Read side-collections with `subscribeChanges` and `useSyncExternalStore`.** `useLiveQuery` hard-fails the `/` prerender. Readiness rides `toArrayWhenReady()`.
- **The tree index mutates in place and notifies synchronously.** Read sibling state before you mutate. Multi-node mutations rebuild the index from the live collection between each step.
- **Read event-time values through the getters** (`getTreeIndex()`, `getViewRootId()`, `getViewIsHidden()`, `getViewFilter()`). Write view state in effects.
- **`createAuth(env, requestOrigin)` is per request**, never a module singleton. The D1 binding exists only inside `fetch`.
- **The `_shell.html` to `index.html` copy is load-bearing.** SPA mode emits `_shell.html`; Static Assets serves `index.html`.
- **DO write atomicity is `ctx.storage.transactionSync()`.** Row writes, the seq bump, and the changelog go in one transaction. Frames broadcast only after a durable commit.
- **Entitlement reads never call Stripe.** `getPlan(userId, env)` is one D1 query on `referenceId = user.id`, `status IN ('active','trialing')`. Free tier is no row. An operator-comped user is a hand-inserted active row with no Stripe ids.
- **Keep the founding seat cap in `getCheckoutSessionParams`**, at checkout-creation time. Webhooks and `subscription.list()` resolve plans from that same list.
- **Reference prices by lookup key**, not price-id env vars. `scripts/stripe-setup.ts` warns on mismatch and does not edit.
- **A node renders in three places**, with three separate keymaps: `OutlineRow` / `useBulletKeymap`, `ZoomedTitle`'s inline `useHotkeys`, and `MiniNodeEditor`. Delegated pointers, `/`, and caret menus already reach all three. Keymaps and slots do not.
- **`el.textContent` is not the source.** Folding tokens render `data-src` atoms. Read with `readSource(el)`. Caret offsets are SOURCE offsets through `getCaretOffset` / `setCaretOffset`.
- **contentEditable text sync is manual.** Stored text writes to the DOM only when it differs.
- **`OutlineEditor` and `SwitcherDialog` carry `"use no memo"`.** Keep the hand-tuned `memo` / `useMemo` on the contentEditable hot path.
- **The visible-neighbor walk must mirror render visibility.**
- **The `refs` registry maps a node id to its contentEditable span**; the zoomed title registers under `rootId`.
- **Assert nesting through `data-parent-id` and `data-depth`.** The windowed list is flat.
- **`rootId` is route-owned.** `routes/index.tsx` gives `null`; `routes/$nodeId.tsx` gives `nodeId`. The editor remounts per zoom via `key={nodeId}`.
- **The typed-error channel in Effect is the error model.** Read Effect v4 from `bunx opensrc path Effect-TS/effect-smol` (its `AGENTS.md` and `.patterns/` first), never `node_modules/effect/`. Search that cache with `grep` or Read. App code imports `effect` from npm.
- **`kv-api.ts` must keep throwing.** TanStack DB rolls back on throw.
- **e2e runs on its own Vite server on port 3210.** Kill a zombie or set `E2E_PORT`. For a caret, set the Selection range directly. `toHaveText` normalizes whitespace.
- **A perf guard asserts a countable invariant**, never a wall clock.
- **When you add an MCP tool, update the ordered tool-name list in `worker/mcp.test.ts`.**
- **`src/routeTree.gen.ts` is generated.** Never hand-edit or reformat it.
- **Every PR adds a changeset fragment.** `bunx changeset --empty` when the PR is not news. `bun run release` is the only way to bump the version.
- **After `bun add` of a React-importing package, clear `node_modules/.vite`** if the dev server dies on an invalid hook call.
- **The Codex app rewrites `.codex/environments/environment.toml`** and drops comments. Put no load-bearing explanation there.
- **`HANDOFF.md` is branch-local.** Commit it on the branch; delete it in the shipping PR. It must never reach `main`.
- **Local dev is `bun run cf:dev` on port 8787.** `bun run dev` on :3000 has a broken database on Cam's machine.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cameronapak/dotflowy](https://github.com/cameronapak/dotflowy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
