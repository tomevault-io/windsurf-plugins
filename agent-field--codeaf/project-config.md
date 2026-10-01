---
trigger: always_on
description: The product is `codeaf`, lowercase, and has had that one spelling since
---

# Working in this repo

## The name

The product is `codeaf`, lowercase, and has had that one spelling since
2026-09-14 — sentence starts, titles, release names, the wordmark and `--help`
included. `CodeAF`, `Codeaf` and `CODEAF` outside an environment variable's name
are not spellings of it.

`internal/namelaw` is the gate. It reads every Go source under `cmd`, `internal`
and `bench`, the build and workflow and script surfaces, both manual corpora,
the session prompts and the documents an agent is pointed at, and it fails the
pull request naming the file and the line that brought a retired spelling back.

## The old name, and the three places it is still allowed

codeaf was called `aforge` until 2026-09-14, in a repository named `aforge-v2`;
`openaf` was a planned name that never shipped and never named a release. A
memory older than that date will spell both, and so will a machine that has been
running this program for a while.

Three places may still say them, and nothing else may:

- **The record.** `CHANGELOG.md`, `docs/changes/`, `docs/design/`, the captured
  screen frames, `bench-results/`, `audit-notes/` and everything under a
  `testdata` directory say what was true on the day they were written.
- **The compatibility seams.** On first start codeaf tries to adopt `~/.aforge`
  into `~/.codeaf` and leave a link behind; if the move itself fails, it carries
  on reading the old folder. An `AFORGE_*` variable is still read when its
  `CODEAF_*` spelling is unset or empty, and `.aforge-v3/config.json` in a
  repository is still read. Persisted and on-the-wire identifiers permanently
  keep their former bytes so old and new builds continue to communicate. Every line
  that has to spell the old name for one of those reasons carries the marker
  comment `legacy-name`, which is the ONLY way a live Go or shell line is
  allowed to say it.
- **The one page that explains it.** A Markdown section whose `## ` heading
  contains the words *old name* — `internal/manual/chat/starting-codeaf.md` has
  it — is where a person who asks "what happened to aforge?" is answered.

A `CodeAF` in this org's Slack and benchmarks is a DIFFERENT PROGRAM (the
swe-pro coding harness, whose variables carry `KNOB`); ours never do.

## Which surface is which

One chat surface lives here, beside the resident that shares its binary and the
component library it draws with. Getting this wrong wastes a whole recon pass, so check
before you read.

| Path | What it is |
| --- | --- |
| `internal/tui3` | **v3 — the live surface.** Entry `cmd/codeaf/chatv3.go`. Bare `codeaf` and `codeaf chat` both open it. |
| `internal/session` | **the v3 engine** — the agent, the turn loop, the toolbelt, tasks. |
| `internal/tui2` | REMOVED as a surface on 2026-08-31, and its compositor, its `blocks` engine and its model picker followed. What remains (`tokens`, `prose`, `reltime`, and `modelui`'s model words) is the shared component library v3 draws with. |
| `internal/head`, `internal/resident` | the v1 **resident** — a different product in the same binary. |

v3 is a **session you sit in front of**. The resident is an employee that keeps working
while the terminal is closed. They share a repository and almost nothing else — do not
carry vocabulary or assumptions between them.

`docs/DESIGN-LANGUAGE.md` is the visual north star: restrained, dim telemetry, no borders.

## Branches — where work goes

`dev` is the trunk and the default branch. Five rules, and they are here rather
than only in `docs/rules/` because they are the ones that must never be looked up:

- **Branch off `dev`, and open the pull request against `dev`.** `main` is the
  release pointer, not a place feature work lands.
- **Never push directly to `dev`, `staging` or `main`, and never force-push any
  of the three.** Promotion is the deliberate fast-forward below.
- **`staging` and `main` move by fast-forward onto tested `dev` history** —
  `git push origin <sha>:staging`, then `git push origin <sha>:main`, never a merge.
  `staging` moves by itself every Friday at the Toronto 17:00 cutoff through
  Promote to staging; a person still moves `main`.
- **Pushes publish channel builds.** `dev` and `staging` publish their named
  channels; `main` publishes an rc. A person cuts stable by dispatching `Release`
  on `main`. The workflow refuses rc or stable commits not already on `staging`,
  and staging commits not already on `dev`.
- **Every pull request carries a change entry** in `docs/changes/unreleased/` —
  `make changelog-new PR=<n> KIND=<kind> SLUG=<slug>`, and the `check` job
  demands it.

The pull-request gate into `dev` is deliberately light — build, vet, the packed
corpora, the change entry, the manual law, a few minutes. The whole suite runs on the way into
`staging` and nightly against `dev`. So **`dev` is where things are allowed to be
briefly wrong**, which is the trade that keeps it fast, and the reason a commit
soaks on `dev` for a couple of days before anyone promotes it.

**AND IF YOUR MEMORY OF THIS REPOSITORY IS OLDER THAN A FEW DAYS, READ
`docs/changes/unreleased/` BEFORE ACTING ON IT.** That is what those entries are
for, and it is the one thing `git log` cannot tell you. They do not say what

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agent-Field/CodeAF](https://github.com/Agent-Field/CodeAF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
