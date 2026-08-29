---
trigger: always_on
description: **This file carries rules, not rationale, in as few words as leave them unambiguous.** No incident narratives, no defence of a rejected alternative, no example that only re-illustrates the rule above it. Cut every hedge, restatement and connective carrying no constraint.
---

# Agent Guidelines

**This file carries rules, not rationale, in as few words as leave them unambiguous.** No incident narratives, no defence of a rejected alternative, no example that only re-illustrates the rule above it. Cut every hedge, restatement and connective carrying no constraint.

**Name nothing you do not have to.** A file, function or constant earns a mention only when the rule cannot be obeyed without it - you must type it or grep for it to comply. A name that merely shows the rule is real, or points at the code implementing it, is rationale in a code font: delete it and check the rule still stands. Say a rule **once**, in the one place that covers its audience.

**Where a piece of writing goes** - decided by who needs it and when, not by what it is about:

| Writing | Home |
|---|---|
| A rule that binds anyone, or that someone could break without knowing they were in that territory | **this file** |
| The how-to for one kind of work - authoring an adapter, writing a migration | a `.dev/` guide, summarized here as its trip-wires plus a link |
| What the system *does* - data model, run pipeline, mechanisms | `.dev/architecture.md` |
| Anything a Hezo user or operator reads | `docs/` |

**A new specialized area is born as a `.dev/` guide, not as a new section here.** This file keeps only the rules that bind someone who does not yet know they are in that territory. Link a new guide from its section and add it to the map below; **Mirrored surfaces** already carries the row that covers guides added, renamed or removed.

**This file has a byte budget, enforced by `agents-md-budget.test.ts`.** When it fails, cut an entry down or move detail to a guide. Raising the number is a deliberate call to argue for in the commit, not a way to make the test pass.

## The `.dev/` map

The rules are here; the detail is there. Prefer reading the guide over rediscovering it.

| Doing this | Guide |
|---|---|
| Anything - what the system does and why | `architecture.md` |
| Writing or changing a test | `writing-tests.md` |
| Running the suite, reading CI, chasing one failure | `ci-and-commands.md` |
| Writing a migration | `writing-migrations.md` |
| Writing agent-facing prose, or authoring a marketplace team | `writing-agent-prompts.md` |
| Translating a string | `writing-translations.md` |
| Changing how a run is judged, delivered or priced | `agent-run-hooks.md` |
| Adding a container backend | `adding-a-container-backend.md` |
| Adding a chat channel | `adding-a-chat-channel.md` |
| Working around the Bun runtime | `bun-issues.md` |
| Looking up where a helper lives | `seam-registry.md` |
| Checking what else a change must touch | `mirrored-surfaces.md` |

Plus point-in-time decision notes and measurements, not rules: `hosted-architecture.md`, `microvm-assessment.md`, `target-audiences.md`, `container-backend-cost-comparison.md`, `mcp-cli-efficiency.md`, `hezo-cloud-requirements.md`.

## Commands

`bun run test` (add `--skip-browser` to drop Playwright, `--pattern` to narrow), `build`, `check`, `check:fix`, `typecheck`, `dev`, `release`. Bundle steps live in the server package and are invoked from there.

- **CI is the canonical check, not a local full run.** A dev box runs the suites serially against one database and fails some for reasons unrelated to your change. Iterate on a subset, keep `typecheck` in the loop, and let CI answer.
- **A run needing a live account or a paid key never runs in CI, and bills real money when you run it.** Supply only the credentials you mean to spend.
- **A required check names a rollup, never a bare matrix job** - a sharded job's name is not stable, so requiring it silently stops gating when the shard count changes.
- **No required check may need a write-scoped token**, or every fork PR fails on a permission it cannot be granted.
- **A suite needing the agent image runs in the container tier**, not on the general shards, which do not build it.

Flags, CI topology, running one file, diagnosing a failure: `.dev/ci-and-commands.md`.

## Layout

- `agents/<template>/*.md` - the single source of truth for agent system prompts. Marketplace teams compile from these plus a roster file; regenerate and commit the generated JSON in the same change.
- `skills/<slug>.md` - the default global skills. Domain-neutral, self-contained, short description.
- `.dev/` - internal engineering docs, and the home for the rationale this file omits.
- `docs/` - user-facing documentation. Some of it is generated; never hand-edit a generated page.

**Where guidance goes - pick by reach.** State a rule once, in the highest-reaching surface that covers its audience:

- Every agent, including future hires, on every run → the shared instructions.
- A subset of seeded roles → a shared partial.
- One role → that role's own prose.

A role doc never restates the shared instructions, and a responsibilities list never restates the workflow below it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hezo-ai/hezo](https://github.com/hezo-ai/hezo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
