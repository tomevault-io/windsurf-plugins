---
trigger: always_on
description: Guide for AI coding agents working in this repo. Read this first, then the
---

# AI Agent Instructions for monid-ai/monid

Guide for AI coding agents working in this repo. Read this first, then the
relevant `openspec/changes/*/design.md` before touching code.

## What this repo is

The open connector standard for [Monid](https://monid.ai). Connectors are
authored in TypeScript (`defineProvider` / `defineEndpoint`: zod schemas + a few
small, closed-term functions), compiled into inert JSON **EndpointDocs** + a
content-addressed **fnTable**, and executed by a small generic engine. Two
released artifacts:

1. `engine/` — the generic connector engine (`@monid/connector-engine`).
2. `.output/catalog.json` — the compiled connector bundle (built by CI, never
   committed).

The legacy imperative provider adaptors live in the sibling repo
`monid-services` (`services/shared/providers/adaptors/*`); connectors are being
migrated here change-by-change.

## Structure

```
connectors/<name>/            # provider.ts + endpoints/<e>/{endpoint.ts, schema/, endpoint.test.ts, fixtures/}
                              #   + resources/<r>/resource.ts for OWNED billable things (saperly/phone-number)
engine/                       # load -> link -> execute; transports; host ABI (ctx.utils)
shared/core                   # THE contract: def/doc/hook/bundle zod schemas, presets
shared/compiler               # pure Def -> Doc mapping, fn normalization + interning
shared/testing                # testSealedUnit / runEndpoint, fixture record+replay
shared/{logging,app-config}   # logger + layered config
scripts/                      # CLI entrypoints (compile, run, catalog, record, version-check)
openspec/                     # spec-driven changes; decision record in changes/*/design.md
config.yml                    # schema.*/compiler.* = CONTRACT (no env overrides); engine/scripts = tooling
```

## Commands

```bash
deno task check && deno task test    # types + replay tests (zero network)
deno task test:live                  # live tests; auto-skip without <PROVIDER>_CREDENTIALS_<FIELD>
deno task engine:run 'exa#search' --body '{...}'   # JIT-compile + execute one endpoint
deno task catalog providers|endpoints|inspect <id>
deno task record <id> ...            # record real fixtures (headers dropped)
deno task compiler:compile           # full-repo deterministic compile
deno task version:check              # semver gate for ABI/format changes
deno task apify:scaffold <actorId>   # authoring-time actor input-schema scaffold
```

## Core invariants (do not regress)

- **Determinism**: the compiled bundle is a pure function of repo content.
  `schema.*`/`compiler.*` config loads override-free; compile iterates sorted;
  hashing is RFC 8785. Double-compile must be byte-identical.
- **Closed-term fns**: hook functions have no imports/captures (TS-AST linted;
  whitelisted pure globals only). All IO flows through the engine's ONE
  transport port — the pure hooks do no IO at all; the three lifecycle hooks are
  effectful-by-capability via `utils.http`/`utils.request` (auth injected at
  egress ONLY for same-origin targets — D16; fns never see credentials).
  Cosmetic edits must not change a fn hash (normalization guarantees this).
- **Hooks (nine)**: six PURE — `auth.inject`, `input.toRequest`,
  `usage.consolidate`, `usage.estimate` (pre-run cost, no IO),
  `output.fromResponse`, `output.fromError` — plus the EFFECTFUL lifecycle
  family — `lifecycle.start`/`poll`/`stop` (async run protocol; monid-services
  `runLifecycle`-shaped). One fallback rule: endpoint ?? provider ?? config
  default, leaf-wise, closest wins.
- **Async protocol**: `request` stays REQUIRED and is DATA into the lifecycle
  (`ctx.data.request`); `lifecycle.start` (when present) replaces the engine's
  declarative execution and returns `RUNNING{state?}` |
  `COMPLETED{httpStatus, output, state?}` (RunKind, UPPERCASE) — WHOLE-STATE
  semantics: a present `state` IS the complete next fn-state (replaces
  wholesale), an absent one carries the previous forward (no field merge — D21).
  State is STRUCTURED (`zRunState`): fn-owned `externalRunId`/`stage`/`data`
  (ids + billing signals; typed per doc via `lifecycle.state` → `stateSchema`) +
  ENGINE-owned `timing` (the ClickHouse provider slices; `RunCompleted.timing`
  reports at settle, sync runs included) — engine-capped
  (`schema.state_max_bytes`). `timeouts.pollMs` is the cadence default, per-tick
  `pollAfterMs` overrides. Every doc floors at `schema.fn_abi_since`.
- **The def IS the rate card (D26 — reverses D18)**: `usage.model` declares the
  billing ALGEBRA per endpoint — LEAF (`FREE` never-bills / `PER_CALL` flat /
  `PER_UNIT` metered, optional block-rate `every`, default 1) and AND
  (`COMPOSITE` with components keyed by OUR snake_case ids; the vendor's native
  spelling, when it differs, is the line's `vendor` FIELD — the join is
  `vendor ?? id`) — and every billable line pins `consumes: {credit, amount}`
  against a credit system declared in `usage.credits` (resolved KEY-WISE,
  endpoint over provider — the pool SET is a provider-wide fact: a provider
  declares every pool its account meters, single-pool providers `default`, a
  dollar-priced vendor's pool IS dollars, pdl one per `x-call-credits-type`;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [monid-ai/monid](https://github.com/monid-ai/monid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
