---
trigger: always_on
description: Visual design tool for Claude — compose HTML/CSS documents with live preview, manage assets, apply brand chartes, export PDF, send via Gmail. MCP server exposed over Streamable HTTP at `POST /mcp`.
---

# Maket

Visual design tool for Claude — compose HTML/CSS documents with live preview, manage assets, apply brand chartes, export PDF, send via Gmail. MCP server exposed over Streamable HTTP at `POST /mcp`.

## Quickstart

```bash
npm run dev       # server --watch + Vite HMR (the one you want 95 % of the time)
npm run dev:watch # server --watch + vite build --watch → public/ (prod-like preview on :24843)
npm run quality   # lint + typecheck + test — same gate as pre-commit
npm run lint:fix  # biome check --write
```

Node `>=22`. **The server must be running before Claude Code connects** (MCP is HTTP-based). The human usually runs it themselves; don't spawn a second dev server.

## Coding principles

These aren't style preferences — they're the invariants the codebase depends on. Breaking any of them reliably breaks boot, tests, or the refactor safety net.

### Dependency injection via Awilix — no exceptions
- Every service, tool pack, and HTTP route is a **factory registered in `bootstrap.ts`**. Consumers resolve by destructured param name. No module-level singletons, no `new`, no direct imports of concrete implementations.
- **Interface first** — declare `export interface Foo { … }` above `createFoo`. Consumers type against the interface; the factory implements it.
- **Factories over classes everywhere** — no `class` keyword in domain code (`DocumentStore` is the lone exception — a legitimate DB wrapper). Private state lives in factory closures.
- **PROXY destructure trap** — every destructured name on `deps` triggers a container lookup. Put optional test overrides on a separate `opts` arg (see `LayoutService`, `PdfService`).
- **Container self-reference** — `bootstrap.ts` registers `container: asValue(container)` so factories that need the container itself (`createMcpRouter` → `mountTools`) resolve it via DI.

### Separation of concerns is hard-enforced
- If you catch yourself adding a runtime field to a persistence type (the old `Document._pending` shape), **that's a new service**. Persistence models hold persistent state; transient state gets its own home. `packages/server/src/services/pending.ts` is the template — factory + interface + private closures + bus-driven side effects.
- **No deprecation shims** — when moving state, remove the old field in the same diff and migrate every call site. "No baby-step migrations."

### Bus emits, listeners propagate
- Services mutate state and emit on the `bus`. Side effects (WS broadcast, toasts) live in `packages/server/index.ts` listeners. **A service never calls `wsRegistry.broadcast` directly.**
- `wsRegistry.broadcast(msg)` accepts typed `WsServerMessage` objects — JSON.stringify happens inside. Don't pre-stringify.

### Complete migrations, four-point checklist on tool renames
Renaming an MCP tool touches four places:
1. The tool file (incl. `declaresTools`)
2. `ACTIVITY_ICONS` in `packages/server/src/routes/mcp.routes.ts`
3. `manifest.json.tools[]`
4. Skill + doc references under `plugin/claude/**`

Boot asserts (1) ↔ (3) and fails fast on drift. (2) and (4) aren't asserted — verify manually.

### Cross-cutting invariants apply at every entry point
Gating logic like `lockGuard` must run in MCP tools, `ws-handler` cases, **AND** bulk UI flows. One missed entry = full bypass. Same for escaping user strings inside `<style>` (`escapeCssValue` + `stripStyleClose`, see `ThumbnailService`).

### Tool output
Always use `text(t, { isError?, next? })` from `packages/server/src/tools/_helpers.ts`. Never build the MCP content envelope inline. Use the `next: string[]` option on business-flow hinges so the agent sees what call to make next.

### Client mirrors server state over WS
- **Zustand = single source of truth.** No Context, no prop-drilled server state. Selectors + `useShallow` for derived shapes.
- **WS for mutations, HTTP only for binaries** (assets, `.maket` bundles). Handlers write the store directly via `useStore.getState()`.
- **Pending queue is the optimistic layer** — `addPending` enqueues, `ack_messages` settles. Don't pre-apply edits to the store.
- **Toasts are server-authored** — `spawnBubble` fires only from the `activity` WS handler.
- **Edit mode is server-authoritative** — any `state` message exits it; don't try to preserve selection across reloads.
- **No `window.prompt` / `window.confirm`** — `InlineNameEditor` for names, `HoldToDelete` or reversible `flagged-delete` for destructive.
- **Popovers inside `SidePanel` portal to `document.body`** (panel's `overflow-hidden` clips). Use `--z-popover` / `--z-modal` tokens.
- **`verbatimModuleSyntax: true`** — `import type` for types.

### Tests
- Co-located: `foo.ts` + `foo.test.ts`. DB tests use `createSQLiteStore(":memory:")` — never touch the filesystem.
- Coverage thresholds in `vitest.config.ts` (core 90 %, services 80 %).

### Gmail is draft-only, read is opt-in
- Maket never calls `users.messages.send` nor `users.drafts.send`. The user reviews in Gmail and sends themselves. No exception.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ng-galien/maket](https://github.com/ng-galien/maket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
