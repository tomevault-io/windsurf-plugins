---
trigger: always_on
description: Canonical instructions for **all** AI coding agents (Codex, Claude Code, Cursor, Copilot, etc.)
---

# Agent Instructions — protspace

Canonical instructions for **all** AI coding agents (Codex, Claude Code, Cursor, Copilot, etc.)
working in this repository; tool-specific files (e.g. `.claude/CLAUDE.md`) import it.

## Spec-driven development with OpenSpec (default workflow)

Plan non-trivial work as an [OpenSpec](https://openspec.dev/) spec **before** writing
implementation code. Proposals, design, spec deltas, and task lists live in
`openspec/changes/<change-name>/`; do **not** save plans, specs, or design docs under `docs/`,
scratch files, or other ad-hoc locations.

- **Use the workflow commands** — or the equivalent OpenSpec skill, or the `openspec` CLI, when
  slash commands are unavailable:
  - `/opsx:propose <idea>` — create a change and its artifacts (proposal, design, specs, tasks)
  - `/opsx:apply` — implement the change's tasks
  - `/opsx:archive` — merge spec deltas into `openspec/specs/` and archive the change; run it as
    the **last commit on the branch, before the merge** — see below
  - `/opsx:explore` — investigate/clarify before committing to a change
- **`openspec/specs/` is the source of truth** for current behavior; read the relevant specs first.
- **Trivial changes** (typo, one-line fix, formatting, dependency bump) do not need a full
  proposal — use judgment.

### Archive before the merge, not after

Run `/opsx:archive` on the branch, commit the result, and let CI go green on that commit.
Deferred to "after the merge" it does not happen — the PR is closed and the branch is gone —
leaving `openspec/specs/` describing behavior the code no longer has, which the next change
reads as current.

Before archiving, tick off `tasks.md` including anything the review added, and reread
`proposal.md` / `design.md` against the final diff: rationale written before a review is often
stale by the end of it, and archiving freezes it.

One-time CLI setup is in [CONTRIBUTING.md](CONTRIBUTING.md#openspec-one-time-per-machine).

## Before committing

Always run `pnpm precommit` before any git commit. It is
`lint-staged && quality && docs:annotations:check && docs:build`:

- ESLint `--fix` and Prettier `--write`, on staged files only (lint-staged)
- TypeScript typecheck, Knip, and Knip dependency validation (`pnpm quality`)
- `docs:annotations:check` — the generated annotation reference must match its source
- `docs:build`, a full VitePress build (a dead internal link fails it)

**It runs no tests at all.** Run `pnpm test` yourself; `pnpm test:e2e` (below) and
`pnpm test:contract` are separate again. It is also JS-only — Python workspace members
have their own CI workflows (see below).

lint-staged only inspects **staged** files, so unstaged work passes `pnpm precommit` and
still fails CI's `format:check` — also run `pnpm format:check` when anything is unstaged.

### A user-visible change is not done until the docs and the notebooks say so

Move these in the same PR — `pnpm precommit` covers none, and all three have shipped stale:

- **The published docs** (`docs/guide/`), for anything reaching a CLI flag, an option default,
  or the bundle format. `docs/guide/annotations.md` is generated, so edit its source instead.
- **The Colab notebooks** (`apps/protspace/notebooks/`), for anything a notebook restates — a
  model list, an install command, a flag. Prettier and ruff's CI paths both skip them, so
  nothing tells you when they drift; import from the package rather than retype, as the prep
  notebook does with `EMBEDDER_MODELS`.
- **`apps/protspace/CLAUDE.md`**, for a new command, test file, or dependency.

Pin a fact that has to live in two places with a test, not a comment asking the next reader to
keep them in step — see `apps/protspace/tests/test_docs_extras_sync.py`.

## End-to-end tests (Playwright)

`e2e.yml` alone drives the real app in a browser; the unit suites run in jsdom, which has no
WebGL. Canvas-dependent wiring — EAT provenance connectors, isolation, dataset swap — is
exercised nowhere else.

It runs nightly on `main`, and on PRs touching the web app, `packages/`, or the root files
those resolve through — `e2e.yml` owns the exact list.

Dispatch it by hand when your change could reach the app by a route that list misses — a
transitive dependency, a shared config, a generated asset — because a wrong `paths:` filter
does not fail, it silently never runs. Not whenever the filter simply didn't match: a PR with
no TS/JS and no root-file changes cannot reach the app, and the run costs ~10 min to confirm
nothing.

```bash
gh workflow run e2e.yml --ref <branch>   # in CI, any branch
pnpm test:e2e                            # locally
```

**Never dismiss a red run as flaky on the strength of local passes.** The regression behind
this rule failed 6/6 in CI and 0/17 locally. Compare against the nightly's history on `main`
(`gh run list --workflow=e2e.yml --event=schedule`), not your machine.

## Python workspace members (uv)

The Python packages are uv workspace members (root `[tool.uv.workspace]`) sharing one root
`uv.lock`. A new **top-level** member needs three things it does not get for free:

- **Its own workflow** in `.github/workflows/` (GitHub runs workflows only from the repo root),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tsenoner/protspace](https://github.com/tsenoner/protspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
