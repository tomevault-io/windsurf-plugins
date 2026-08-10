---
trigger: always_on
description: Local-first sync engine: local SQLite is the read model, writes go through an
---

# Syncular agent instructions

Local-first sync engine: local SQLite is the read model, writes go through an
outbox, servers converge clients. One TS core and one Rust core implement the
same wire protocol, **conformance-locked** against each other. Docs site:
syncular.dev (built from `apps/docs`, tracks main).

This file is the canonical instruction set for coding agents and applies to
human contributors just as much. `CLAUDE.md` is a gitignored symlink to this
file, created by the root `postinstall` script.

Read before non-trivial work: `docs/SPEC.md` (wire truth), `docs/SYQL.md`
(query-language truth), the relevant package README, and `docs/RELEASE.md`
when changing release behavior. Do not revive removed mechanisms or add a
parallel path without current evidence and a specification change.

## Doctrine (enforced)

- **Spec-first**: wire-behavior changes start in SPEC.md; judgment calls get
  codified back into it.
- **No fallback paths.** Loud, precise errors over silent degradation.
- **No timers in tests.** Deterministic flush/readiness helpers only
  (`flushQuerySchedulers`, chained microtasks); a wall-clock sleep is a bug.
- **Cross-core parity**: behavior observable on the wire or through the
  client surface must match TS and Rust. Semantics changes (e.g. window
  completeness, invalidation granularity) require BOTH cores plus a
  conformance catalog scenario (`packages/conformance/src/catalog/`); a
  TS-only change breaks the rust-conformance CI pairing.
- Commit/push: local commits after verification are fine; **push only on
  the maintainer's explicit instruction**.

## Coding rules

- **No unnecessary code artifacts.** Direct code only: no named helpers,
  wrappers, constants, or files for single-use logic. Inline it unless it is
  reused, explicitly requested, or shared between production code and tests.
- **Less is more.** Rewrite an existing component before adding a parallel
  one. Prefer editing existing files over creating new ones.
- **Bugfix restraint.** For bug fixes, first check whether existing code can
  be simplified, reduced, or localized; add new arguments, plumbing, or
  abstractions only when the root cause shows they are necessary. Tests can
  grow freely.
- **Scan for repeats.** After finding a bug, `rg` for the same pattern
  elsewhere in the repo before fixing a single instance.
- **Clean codebase.** Flag obsolete files for removal.
- **Stable error identities.** Errors carry static codes
  (`sync.outbox_incompatible` style); dynamic values (ids, cursors, raw rows)
  belong in structured error details, never interpolated into the message.
- **No `as any` / `as unknown`.** When something fails to typecheck, fix the
  underlying type.
- **Testable change → test.** Add or extend a `bun:test` file next to the
  change. No `mock.module`; use dependency injection or scoped test doubles.
- **Docs and changelog ship with the feature.** A change that alters what
  users see or do updates the docs site (`apps/docs/src/content/*.md`);
  feature-level work also adds an entry to the changelog manifest
  (`apps/docs/src/changelog.mjs`, newest first, linking to the docs page
  that covers it).
- **English everywhere**: code, comments, commit messages, docs.
- **Commits are human.** No AI attributions, `Co-Authored-By` trailers, or
  tool names in commit messages.
- **Leave others' work alone.** Never run destructive git commands
  (`revert`, `reset --hard`, forced checkouts) over changes you did not make.
- **No unrequested publishing.** Deploy, release, and publish scripts run only
  on explicit instruction, like pushes.

## Prose rules for docs

Reader-facing text gets the same review as code.

### Voice

Write as the engineer who built the system, addressing a peer who will
build on it. The register is a good reference manual: declarative,
concrete, unhurried. Every sentence carries a fact, an instruction, or a
consequence; a sentence that carries none is deleted. When something is
uncertain or unmeasured, say so plainly or leave it out.

### Sentence mechanics

- Actor first, active verb: "The server validates whole commits", never
  "Commits are validated". Passive voice only when the actor is unknown
  or irrelevant.
- Name the component doing the work (`typegen`, the outbox, the client
  core). "The system", "the solution", and "functionality" are banned
  subjects.
- One canonical name per concept, reused verbatim across the page.
  Repeating a technical noun is correct; rotating synonyms makes readers
  hunt for a distinction that does not exist.
- Numbers and names over adjectives: "bootstraps 100k rows in 30.7 ms
  warm", never "fast bootstrap". A claim needs a number, a spec section,
  or a code path behind it; otherwise cut it.
- Instructions in the imperative, consequence stated as fact: "Use a
  persistent database path. An in-memory database loses the outbox on
  restart."
- Define a term at first use or link to the page that does.
- Bullets only for genuinely parallel items (options, fields, steps).
  Connected reasoning stays in paragraphs.

### Banned patterns (generated-prose tells; delete on sight)

- **No em-dashes.** Use a colon, semicolon, comma, parentheses, or a
  sentence split. List items are "`thing`: description".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [syncular/syncular](https://github.com/syncular/syncular) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
