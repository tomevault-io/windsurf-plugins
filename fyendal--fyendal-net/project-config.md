---
trigger: always_on
description: Fyendal is a TypeScript/pnpm monorepo for playing Flesh and Blood. Preserve
---

# Fyendal agent guide

Fyendal is a TypeScript/pnpm monorepo for playing Flesh and Blood. Preserve
the boundaries below; they are release invariants, not suggestions.

## Architecture invariants

- Production may run multiple Cloud Run instances. Live sockets and their
  delivery bindings remain local; durable matchmaking, leases, commands, and
  compact cross-instance events live in Postgres. Fan-out polls the event log;
  do not add `LISTEN`/`NOTIFY` or assume an external distributed queue.
- Schema epoch 1 is the initial empty-database baseline. Unexpected application
  tables or another epoch fail with `RESET_REQUIRED`. Future changes append
  immutable migrations; never edit an applied migration.
- `rulesetVersion` is an operator-managed compatibility identifier, not the
  image SHA. Compatible deployments retain it; incompatible persisted-state
  or rules changes require an explicit bump. The explicit cutover deletes
  rooms from another ruleset rather than migrating active games. Cutover is
  fenced; startup never changes the active ruleset implicitly.
- `packages/shared` contains types only. All wire, HTTP-response, `GameView`,
  and replay decoding belongs in `@fyendal/protocol`.
- Parse JSON and database JSON as `unknown`, then decode it exhaustively. Never
  cast untrusted values to runtime types.
- Card scripts observe deeply readonly state and mutate only through
  `ScriptCtx`. Do not create mutable player helpers, engine-runtime imports,
  compatibility adapters, or internal escape hatches.
- Public logs must not reveal identities from hands, face-down arsenal, private
  deck positions, blind choices, or other hidden zones. Use `logPublic`,
  `logPrivate`, or `logForSeats` deliberately.
- Every registered rules limitation has a source annotation and a focused,
  executable `it.fails` scenario.

## Packages and applications

- `packages/shared` — data contracts and stable protocol error codes.
- `packages/protocol` — hand-written exact-key runtime validators with bounds,
  safe-integer checks, nested validation, and depth limits.
- `packages/engine` — deterministic, zero-I/O, card-agnostic rules engine.
  Never reference a printing id, card name, class, or hero here. Read
  `packages/engine/DESIGN.md` before changing rules behavior. Import mechanics
  from their owning modules; do not turn `util.ts` back into a compatibility
  barrel. Production engine modules must remain free of runtime and type-only
  dependency cycles; the architecture test enforces this without an allowlist.
- `packages/cards` — card JSON, functional scripts, vanilla registry, precons,
  and scenario tests. Use `.agents/skills/import-cards/SKILL.md` for imports.
- `apps/server` — HTTP/WebSocket gateway and Postgres stores. `db.ts` owns the
  schema, `persistedState.ts` owns the explicit persisted DTO,
  `roomBroadcaster.ts` owns local fan-out, and `gateway/` owns connection,
  matchmaking, routing, and composition.
- `apps/client` — React/Vite UI and one public Zustand store covering
  connection/session, account/decks, lobby/room, and replay state.

## Internationalization

- React copy uses `react-intl`. Source catalogs live in
  `apps/client/src/i18n/catalogs/{en,zh-Hans}.json`; compiled ICU AST catalogs
  live in `apps/client/src/i18n/compiled/`. Edit both source catalogs and never
  hand-edit compiled output. English is the canonical fallback, and every
  supported locale must have exactly the same keys.
- Message ids are stable, lowercase, dot-separated semantic identifiers. Do
  not encode rendered copy, a printing id, or runtime data in an id. Put
  dynamic content in ICU values; use typed `GameMessage` card, player, and term
  references when the client must resolve visible names safely. A term
  reference composes another catalog message using the enclosing values; use
  it for reusable fragments such as trigger effects instead of duplicating
  every effect as a second log translation.
- Client-only copy uses `useIntl`, `FormattedMessage`, `GameMessageText`, or
  `formatGameMessage`. Do not branch on the current locale or construct
  translated sentences by concatenating fragments.
- The engine, server, and card scripts never import locale catalogs or render a
  locale. Wire-visible game copy retains a deterministic English fallback and
  carries locale-neutral `GameMessage` metadata. This keeps old clients,
  persisted decisions, replay diagnostics, and newer catalogs compatible.
- Card-script decisions use the helpers in
  `packages/cards/src/scripts/shared-helpers.ts`: `decisionPrompt` for every
  player-facing prompt, `decisionMessage` for semantic option labels, and
  `commonOptionMessages` for shared fixed choices. Option values delivered to
  `onChoose` are stable engine values and must never be translated. Card choices
  may remain ids because the client renders their visible card names.
- Player-facing card logs use `localizedLog` or `localizedCardLog`. Choose
  `logPublic`, `logPrivate`, or `logForSeats` independently of localization;
  neither fallback text nor semantic values may reveal hidden information.
- Engine `logPublic` producers must pass a semantic `GameLogPayload`, never a
  raw player-facing string. Server-generated game-log entries (such as undo or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fyendal/fyendal.net](https://github.com/Fyendal/fyendal.net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
