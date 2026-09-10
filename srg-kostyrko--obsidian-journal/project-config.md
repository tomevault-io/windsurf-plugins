---
trigger: always_on
description: Working notes for AI agents and new contributors. Two parts: where the
---

# CLAUDE.md

Working notes for AI agents and new contributors. Two parts: where the
authoritative docs live, and the traps that none of them cover.

## Where things are documented

Each subject below has exactly one owner. Read the owner — this file does not
restate it.

| Document                                                               | Owns                                                                                             |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| [`CONTEXT.md`](CONTEXT.md)                                             | domain vocabulary — periods, journals, shelves, decorations                                      |
| [`docs/architecture.md`](docs/architecture.md)                         | code layout, DI, `Result`/`Option`, dates and union dispatch, schemas, i18n, test file locations |
| [`docs/unit-testing-strategy.md`](docs/unit-testing-strategy.md)       | the unit and component suite — tiers, `testContainer`, fixtures, assertions, lint rules          |
| [`docs/e2e-testing-strategy.md`](docs/e2e-testing-strategy.md)         | the e2e layer — runner, fixtures, selectors, execution model                                     |
| [`docs/i18n-glossary.md`](docs/i18n-glossary.md)                       | translation terms, and the `check:i18n` rules that `scripts/check-i18n-glossary.mjs` enforces    |
| [`docs/2026-07-13-ux-text-audit.md`](docs/2026-07-13-ux-text-audit.md) | user-facing copy style — sentence case, error grammar, en-US                                     |
| [`docs/manual-testing-checklist.md`](docs/manual-testing-checklist.md) | the manual verification pass                                                                     |
| [`docs/plugin-harvest-list.md`](docs/plugin-harvest-list.md)           | the standing queue of plugins to mine for ideas, and what has already been ruled out             |
| [`docs/releasing.md`](docs/releasing.md)                               | how a version reaches the community store                                                        |
| [`docs/plugin-api.md`](docs/plugin-api.md)                             | the plugin-facing API — its surface, stability policy, and the npm package                       |
| [`CONTRIBUTING.md`](CONTRIBUTING.md)                                   | setup, quality gates, commit and PR conventions                                                  |

**If a rule belongs to one of those documents, it goes there, not here.** This
file carries only what has no other home.

## Specs and plans

Design specs and implementation plans are written to `.superpowers/specs/` and
`.superpowers/plans/`, which are git-ignored. They are working artifacts, not
repository content — `docs/superpowers/` was retired on 2026-08-13 and is
ignored to keep it retired. Earlier specs remain in git history:

```bash
SHA=$(git log --all --diff-filter=D --format=%H -1 -- docs/superpowers)
git ls-tree -r --name-only "$SHA^" -- docs/superpowers   # 202 files
git show "$SHA^:docs/superpowers/specs/<name>.md"
```

`--all` is load-bearing: without it, history simplification prunes the deleting
commit and the lookup silently returns nothing, reading as "unrecoverable".

`src/_old-code/` is gone the same way, and it was what nearly every deleted spec
cited for "v2 did X". Since the v3 merge `main` **is** v3 — the last v2 release
is the `2.1.10` tag, so check such a claim there:

```bash
git show 2.1.10:src/journals/journal.ts
git grep <term> 2.1.10 -- src
```

## Standing rules

Project-wide decisions with no other home. They decide what counts as a bug.

- **v3 has shipped.** `3.0.0` was tagged on 2026-08-14 — `manifest.json` reads
  3.0.0 and `CHANGELOG.md` carries a dated `[3.0.0]` section. People now run v3,
  so a shape written by 3.0.x is data someone owns: a gap found in it needs a
  repair path, not a documented note. This inverts the pre-release rule, and
  decisions in git history that accepted a v3-era gap were made under that older
  rule — they are not precedent for a gap found today.
- **A missing v2 behavior is a bug report, not a settled decision.** The port is
  finished and `src/_old-code/` is gone, but v2 users upgrade into v3 and report
  what they lost. Check the claim against the v2 source at the `2.1.10` tag
  before answering, and treat a real gap as a defect unless it appears under
  "Deliberate non-bugs" below. One deviation _was_ opted into: the v1 → v2
  migration runs non-interactively in v3, recorded in the migration section of
  [`docs/manual-testing-checklist.md`](docs/manual-testing-checklist.md).
  Settled decisions that merely _read_ as regressions are under "Deliberate
  non-bugs" below; none of those is a dropped v2 feature.

## Traps

Behavior that is expensive to rediscover and invisible in the code that depends
on it.

### Boot and lifecycle

- A boot-time whole-vault note walk must wait for layout-ready (the file list)
  **and** for every note to resolve in `metadataCache`, re-checking on each
  `resolved` batch. Waiting on only one skips notes while still clearing its
  markers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srg-kostyrko/obsidian-journal](https://github.com/srg-kostyrko/obsidian-journal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
