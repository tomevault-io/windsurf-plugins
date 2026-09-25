---
trigger: always_on
description: House Plan is one HACS package with two parts plus a demo harness:
---

# AGENTS.md

House Plan is one HACS package with two parts plus a demo harness:

- **Lovelace card** (`src/`, TypeScript + Lit) — the primary product, bundled to
  the entry, manifest and hashed chunks under `dist/`.
- **Storage integration** (`custom_components/houseplan/`, Python) — the Home Assistant backend.
- **Demo harness** (`demo/`) — a self-contained Playwright page (`demo/srv/demo.html`) that renders the card against a fake `hass`, used for screenshots and the `smoke_*.mjs` end-to-end suite.

## Read this first

**`docs/SCOPE.md` before anything else.** It was fixed with the owner and states
its own authority: features are built, improved and accepted **only** if they
serve a job listed there. It carries the mission, the three personas, the core
user jobs and the out-of-scope list.

Its central consequence: **View mode is the product for two of the three
personas.** Editors are admin-only tools and must never leak interactions into
View.

For work that changes visible behaviour, also read `docs/USER-GUIDE.ru.md` —
interface wording comes from there and is not invented, or the UI starts speaking
developer.

Then `PROCESS.md` (the full process), `docs/STATUS.md` (where the release line
is), and for non-trivial changes `docs/ARCHITECTURE.md` plus the canonical
document of the subsystem you touch: `SUN.md`, `LIGHT.md`, `CANVAS.md`,
`WALL-THICKNESS.md`, `UX-MODES.md`, `CONFIG-COMPATIBILITY.md`,
`TOUCH-SUPPORT.md`.

Standard commands live in `package.json` scripts, `CONTRIBUTING.md` and
`docs/DEVELOPMENT.md`.

## Canonical backlog and status

[GitHub Issues](https://github.com/Matysh/houseplan-card/issues) are the canonical
task records: problem, scope, acceptance criteria and discussion.

**Status lives in labels:** `S1-new`, `S2-analysis`, `S3-spec`, `S4-spec-review`,
`S5-ready`, `S6-in-progress`, `S7-code-review`, `S8-merged`, plus `blocked` on top
of a status and `rejected` on a closed issue. A product issue in flight carries
exactly one `S*` label. An infrastructure-only issue is the deliberate exception:
it may carry no `S*` label while being implemented, enters the common flow at
`S7-code-review`, and from then on carries exactly one. Labels are the whole of
it: GitHub Projects is no longer used.

**The light track is the default, not a shortcut** (owner's decision 2026-08-27,
issue #338). `small` — the spec lives in the issue body and its review is a
comment. Analysis names the `small` criterion the task *fails* when it takes the
full track; "ordinary track" without a named criterion is not a justification.
The threshold itself did not move — only which side carries the proof. The full
track stays what it was for geometry, config migrations and public contracts,
where a criterion is broken plainly and saying which one is easy.

`trivial` — the short track: no spec stage at all, `S2-analysis` straight to
`S5-ready`, with the AC written into the issue body first. `trivial` requires a bug confined to one surface with no new UX
contract, no migration, no i18n, no perf or touch impact, at most three checkable
AC, **and expected behaviour already on record** — nothing left to decide. Code
review is never skipped on either track: it checks scope, risks and the evidence
from executed tests, but does not stand in for executing them.
`PROCESS.md` §5 and §5.1 hold the criteria.

An issue filed by an outsider is worked exactly like one of the owner's own, once
the owner has decided to take it. For product work the check sits **at the
entrance**, not on every step: the first status label admits it to the flow. For
infrastructure work an explicit assignment by the owner is the entrance, and the
issue may remain without an `S*` label until its first `S7-code-review`. In either
case, **who filed it stops mattering** once the owner has admitted it.

Applying that first label *is* the owner's explicit decision, and the platform
already guarantees it — only someone with write access can label. The earlier rule
made outside reports be refiled as the owner's own issues, which turned out to be
work for nothing: on #123 the spec was already written by the time the guard
refused.

Specs, audits and ADRs may live under `docs/`, but must link to their issue and
must not become a parallel task list. When repository documentation disagrees with
Issues, the issue wins.

## Rule #1

> Changing product code without an issue is forbidden. Code changes only when the
> issue exists and sits in "Ready for development" or later.

Check before touching product code:

```
gh issue view <NN> --repo Matysh/houseplan-card --json number,state,labels
```

The label must be one of `S5-ready`, `S6-in-progress`, `S7-code-review`. Anything
else — refuse and say why. "Issue #83 is in `S2-analysis`, code is off limits.
Start with the spec?" is the correct answer, not a smaller patch.

## Change classes

| Class | Paths | Issue required |
|---|---|---|
| **A — product** | `src/**`, `custom_components/houseplan/**/*.py`, `manifest.json`, `hacs.json`, i18n, `custom_components/**/translations/**` | yes |
| **B — gates and tooling** | `test/**`, `tests_backend/**`, `demo/**`, `scripts/**`, `.github/workflows/**`, `rollup.config.mjs`, `tsconfig*.json` | yes; may reuse the issue it covers |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Matysh/houseplan-card](https://github.com/Matysh/houseplan-card) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
