---
trigger: always_on
description: Harbor is the Spaces server: orgs, spaces, members, an append-only log, three faces. This file is the mechanics — where things live, the invariants, the slice a capability cuts through, how to run and ship. What Spaces *is* and why is [SPEC.md](./SPEC.md); what the wire *means* is [CONTRACT.md](./CONTRACT.md). One kind of fact per document; link, never restate.
---

# Harbor — how it is built

Harbor is the Spaces server: orgs, spaces, members, an append-only log, three faces. This file is the mechanics — where things live, the invariants, the slice a capability cuts through, how to run and ship. What Spaces *is* and why is [SPEC.md](./SPEC.md); what the wire *means* is [CONTRACT.md](./CONTRACT.md). One kind of fact per document; link, never restate.

## Layout

Two pnpm workspace packages under `packages/`:

- **`protocol/`** — `@rowboat/spaces-protocol`, the contract: zod schemas imported by the server *and* the app, so drift is structurally impossible. `core.ts` (the objects), `ids.ts` (ids, the link grammar and its one parser), `changeset.ts`, `events.ts` (`SpaceEvent` and the live frames), `api.ts` (`routes`), `mcp.ts` (`mcpTools`), `mentions.ts`, `search.ts`, `invite.ts`, `errors.ts`, `fixtures/merge/` (the golden merge cases every engine must pass).
- **`server/`** — `@rowboat/harbor`:

| `src/` | Owns |
|---|---|
| `core/kernel.ts` | store, hub, org, the read-only knob, the space lock with its publish-after-commit outbox, `append` / `nextOffset` / `appendNext`, `requireSpace` / `requireReadableSpace` / `requireMember`, `guardWrite`, `attributionOf` |
| `core/spaces.ts` | spaces, direct messages, invites and the bind ceremony, the roster, `me`, agent members (`createAgent`), push registration, the read-gated replay and membership-gated live relays |
| `core/agents.ts` | agent members' owners and keys: add an agent, list the ones a member manages, create and revoke keys |
| `core/assets.ts` | assets by id, versions, the change log, blobs, history, diff |
| `core/feed.ts` | messages, threads, topics, reactions, polls, search, mention stamps and their backfill |
| `core/read-state.ts` | read marks, follows, unread, Activity, read-all |
| `service.ts` | `HarborService`, the facade: one delegate per public method, `org` / `readOnly` accessors |
| `policy.ts` | who may do what — pure decisions over facts the core loads; `enforce` throws |
| `store.ts`, `pg-store.ts` | the data boundary and its one driver; `PgStore.transaction(fn)` for an org-level all-or-nothing write the caller shares (`directory.ts`); `sql.ts` (node-postgres), `sql-pglite.ts` (Postgres in-process) |
| `migrations.ts` | the append-only schema ladder |
| `http.ts`, `ws.ts`, `mcp.ts` | the three faces; `origin.ts` (the public origin behind the proxy) |
| `auth.ts`, `auth-oidc.ts` | the drivers, `bindAuth` / `OrgAuth` (which resolves agent keys ahead of any driver), `authenticateRequest`, the RFC 9728 helpers; `consent.ts` (the login page); `agent-keys.ts` (minting and hashing an agent key) |
| `runtime.ts` | `buildOrgRuntime` — the one assembly of an org |
| `server.ts`, `main.ts` | `startHarbor` (one org) and the dev seed; the binary (dev, or `HARBOR_MODE=deployment`) |
| `deployment.ts`, `directory.ts`, `apex.ts` | many orgs from one process: host → org runtime; the org directory; the apex face (create org, my orgs) |
| `notify.ts`, `push.ts` | the one notification decision; Expo delivery |
| `hub.ts`, `blobs*.ts`, `mime.ts`, `merge.ts`, `search.ts`, `mentions-backfill.ts` | in-process fan-out, blob drivers, sniffing, the three-way merge, query parsing, the mentions backfill |
| `stats.ts`, `internal.ts` | the live-load counters (connections, subscriptions, frames per minute by kind, deliveries) and the operator face that reads them, `GET /internal/stats` behind `HARBOR_INTERNAL_KEY` |

`test/` has one file per feature, every one on in-process Postgres. `helpers.ts` gives `startTestHarbor` (a harbor over a fresh database, closed with it), `restClient`, `agentClient`, `liveClient`, `startFakeAs` (a fake authorization server: discovery, JWKS, minted JWTs). `day-in-the-life.test.ts` is spec §11 as code; `mcp-parity.test.ts` proves the agent face; `policy.test.ts` pins every rule without a store.

## Invariants

- **One core, three doors.** The faces hold `{ service, auth: OrgAuth }` and never the store. Rowboat's own agent uses the same MCP tools as any agent; there is no privileged path.
- **Rules live in `policy.ts`.** Every question of the form "may this actor do this to this space or message" is a pure decision there; the core loads the facts and asks; no face decides anything.
- **Reading and acting have separate gates (2026-09-23, spec §5).** `requireReadableSpace` loads the space, membership, and org member for `canReadSpace`; only shared open spaces admit nonmembers. `requireMember` and `lockedAs` retain the membership-only `canAccessSpace` rule. `requireOrgMember` guards browse/self-join. Durable events use the transaction outbox, never publish an uncommitted write.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rowboatlabs/rowboat](https://github.com/rowboatlabs/rowboat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
