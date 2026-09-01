---
trigger: always_on
description: validates only the storage boot slice (agent-base's `config/boot.js`: db+log), so a
---

# CLAUDE.md — conventions for this repo

Guidance for any Claude Code session working in `swampratnz/community-agent`.

## What this is

A TypeScript/Node service (the "NZ Claude Community" agent) that bridges a
Discord server and a WhatsApp number to a Claude Agent SDK agent with
persistent Postgres + pgvector memory and a gated three-tier RBAC model. Start
with `README.md`, then `docs/ARCHITECTURE.md` and `docs/SECURITY.md`.

**To find your way around the code, read `docs/agents/`** — a committed context
pack aimed at exactly this situation: `module-map.md` says which module owns
which behaviour (security spine marked), `recipes.md` says what a given kind of
change normally touches and which gate catches a missed file. It exists because
every pipeline worker is a fresh Actions run, i.e. a cold session that would
otherwise re-derive this repo's layout on every single run. Use it instead of a
broad exploration sweep — then read the actual code, because the pack is
orientation and never authority. If it is wrong, fix it in your PR.

## Where the framework lives

The framework is **[`@swampratnz/agent-base`](https://github.com/swampratnz/agent-base)**,
consumed as a package: the agent kernel and prompt spine, the platform
adapters, storage, the router spine, the jobs mechanism, RBAC, config, the
notice-catalogue mechanism, alert/health infra, leaf utils. `src/base/` is GONE
and must stay gone — `npm run imports:check` fails outright if it reappears,
because a local copy forks the package silently. A framework-level fix belongs
upstream and reaches this repo as a version bump.

What is here:

- **`src/module/`** — this deployment's content and wiring: the tool registry
  and its `ToolDef` domain files, prose, personas, skills, the notice pack,
  community jobs and digests, the integrations, its schema fragments, and the
  composition wiring (`routerWiring.ts`, `platforms/factories.ts`,
  `jobs/registry.ts`, `commands.ts`).
- **`src/module/agentModule.ts`** — THE manifest. Every extension point this
  deployment fills, named once, as data. Its `init()` is also boot-fatal on two
  env vars: `DISPLAY_TIMEZONE=Pacific/Auckland` and `DISPLAY_LOCALE=en-NZ`.
  agent-base defaults them to `UTC`/`en-GB` — it cannot know a deployment's
  timezone — so the manifest asserts them rather than let every member-facing
  event time silently re-render an hour out. It throws from `init()`, so the
  failure is a plain `Error` with those two names in it, NOT config's zod
  `Invalid environment configuration` exit. Set both in `.env`.
- **`src/index.ts`** — the composition root: it hands that manifest to
  `createAgent`, then wires adapters, the router and the jobs, and owns
  startup/shutdown ordering. The only file that may compose.
- **`src/migrate.ts`** — `npm run migrate`: base fragments, then this module's.

**Adding an extension point** means exporting the value from the file that owns
the content and naming it in the manifest. Do NOT add a module-scope
`register*()` call, and **never render a `notice()` at module scope** — the
pack is registered by `createAgent`, AFTER every module has been imported, so
an import-time render throws before the process can say why. Tests opt into the
same registrations one slice at a time through `tests/support/register*.ts`.

`createAgent` owns the order and it is not negotiable from a module: plan (a
pure pass that rejects an incomplete or double-claimed composition with the
process untouched) → each module's `init()` → singleton registrations →
additive registrations → readiness probe → migrations → start.

One gap to expect, real today and an upstream fix: the manifest type has no
`configSchema` field, so a new env var is an agent-base change. Mind which
type you are reading — the package exports TWO things called `AgentModule`:
the live one from `createAgent.d.ts`, re-exported as `AgentModuleManifest`
(what `src/module/agentModule.ts` imports, and the one with no
`configSchema`), and an older `module-api/module.d.ts` one that has the field
but is not what `createAgent` takes. (Subpath
exports were the other one; `@swampratnz/agent-base@0.1.1` ships them, so
`@swampratnz/agent-base/<module>.js` resolves straight from the package and the
postinstall shim that used to add them is gone.)

## Build / test / verify

- `npm run typecheck` — must be clean. This now also runs
  `npm run typecheck:tests` (`tsconfig.tests.json`), because the main tsconfig
  covers `src/**` only and `tsx` strips types without checking them, so `tests/`
  went entirely untypechecked — which is how a whole class of test bug survived:
  an injected-`deps` object that omits a field silently falls through to the
  REAL repository function, so a "unit" test quietly queries live Postgres, and
  since `node:test` runs test FILES in parallel those stray reads land on tables
  other files are counting (a source of the cross-file flakiness that reddens
  unrelated PRs). `tests/` has a large backlog of pre-existing type errors, so
  this is an **incremental ratchet**: the config's `include` lists only the test
  files that are clean today, kept alphabetical one-per-line so concurrent PRs
  merge cleanly. Bringing another file to zero and adding it is the unit of

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swampratnz/community-agent](https://github.com/swampratnz/community-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
