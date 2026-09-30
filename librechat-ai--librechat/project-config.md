---
trigger: always_on
description: `AGENTS.md` is the repository's contributor guidance. Backend code lives in `packages/api`
---

# LibreChat contributor and agent guidance

`AGENTS.md` is the repository's contributor guidance. Backend code lives in `packages/api`
(TypeScript) and `packages/data-schemas` (database methods); `api` is legacy Express wiring.
Shared API types and services live in `packages/data-provider`, and the React app lives in
`client` with shared primitives in `packages/client`.

## Branching and pull requests

Normally branch off `dev` and target `dev`; `gh pr create` defaults to `main`, so pass `--base dev`
explicitly. `main` is the released branch, kept as a fast-forward of `dev` and synced as-is — never
open a backport pull request to `main`, because anything merged to `dev` reaches it at the next
sync. Pull requests opened against `main` are retargeted automatically.

**Maintainer-directed canary exception:** Experimental work, or a PR ready to merge after review but
not yet suitable for the next `main` sync, may instead target `canary`. The maintainer may choose
this before work starts or while reviewing a stale PR. For new canary work, branch from the current
`origin/canary` and pass `--base canary` explicitly; do not silently retarget an existing PR or
promote canary code to `dev`. If a stale PR is redirected to canary, first check its base, diff and
reviewed head against current canary; coordinate any rebase or new PR with the maintainer. A PR
explicitly based on `canary` stays there; the main-to-dev retarget workflow does not move it.

Still link related issues in the PR (for example, `Related to #N`) so the work remains
traceable. `Fixes #N` does not close an issue on a `dev` or `canary` merge — GitHub honors closing
keywords only on the default branch. Close resolved issues by hand after merging. Worktrees share
one stash stack, so never use a bare `git stash pop`. Prefer a WIP commit; if a stash is necessary,
apply the specific tagged entry. The `target: main` label and release-bound upstream branches are
exempt from the main-to-dev retarget workflow; do not use either to backport ordinary work.

Write the description for a reader who has not followed the branch: what breaks, what triggers it,
how it behaves after the change, then one or two views of the mechanism — a focused diff, a call
tree, a shallow file tree, or a Mermaid sequence. Keep only what the change carries, and describe
the code as it stands rather than narrating earlier commits or review rounds. Naming the merged
pull request that caused the bug is not the same thing; that is history the reader needs. The
formats and examples live in `.github/pull_request_template.md`.

## Review and completion

Read the inline review threads themselves — a summary comment or notification list omits findings.
Audit each one against the current code, fix what is valid, and reject what is obsolete in a reply
that says why. After each round: focused tests, `npx tsc --noEmit` in every workspace you changed,
push, then request the next review naming the pull request's exact remote head — a clean review of
an earlier head says nothing about what you just pushed, and CI runs on its own clock. Reply on
each resolved thread with the resolving commit and evidence. After two actionable rounds, stop
patching thread by thread and read the subsystem by invariant instead: identity, authorization,
persistence, retry, cancellation, cleanup and mixed-version behavior where relevant. Which reviewer
and what phrase triggers it will change; that the review must cover the exact pushed head will not.

A clean review is one completion signal, not the definition of done. Ship the observable experience
— loading, empty, success, failure, cancellation, retry, restored session — with strings localized,
accessibility intact, defaults and stored data preserved, and no backend capability left without a
frontend entry point. Keep fixes small and test the missed behavior. Report the pushed head, what you
ran locally, CI state, the review result at that head, any finding you rejected with the reasoning,
and checks you could not run.

## Verification

For startup, auth, config, file, or message-loading changes, avoid serial database
reads and reuse loaded request data. Run `npm run lighthouse` before completion:
the CI lane adds 250 ms per Mongo query and checks the visible conversation's LCP.
See [budgets, reproduction and failure diagnosis](e2e/lighthouse/README.md).

A green build is not a typecheck: `packages/api`, `packages/client` and `packages/data-schemas` build
with `tsdown`, which emits without checking types. Run `npx tsc --noEmit` in the workspace you
changed. `packages/client` excludes `*.spec.ts(x)` and `*.test.ts(x)` from typechecking entirely.
`npm run sort-imports` with no arguments rewrites every source root — pass the paths you touched.
Run `npm run static-checks -- --against origin/dev` to reproduce the PR's static checks; use
`npm run static-checks:full` for the slower gates. Fix all formatting, lint, and TypeScript
warnings/errors in the code you change.

## Module boundaries and configuration

All new backend behavior belongs in TypeScript under `packages/api`; database-specific shared logic
belongs in `packages/data-schemas`, and frontend/backend shared API logic belongs in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LibreChat-AI/LibreChat](https://github.com/LibreChat-AI/LibreChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
