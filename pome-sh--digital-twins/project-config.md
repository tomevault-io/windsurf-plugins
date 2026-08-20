---
trigger: always_on
description: > Operational notes for anyone — human or AI agent — working in this repo.
---

# AGENTS.md

> Operational notes for anyone — human or AI agent — working in this repo.

## What this repo is

The open-source Pome twins and the `pome` CLI. The twins are local, resettable
services that answer the same REST, GraphQL, and MCP calls your agent makes in
production (GitHub, Stripe, Slack, Gmail, Linear), each backed by real SQLite
state. The CLI boots a
twin, runs an agent against it, and records the trace for you to inspect.
Evaluation and scoring are hosted features — `pome run --local` records a trace
only, no local scoring. The Pome platform (evaluation, simulation,
observability) is at https://pome.sh. Apache-2.0.

## Docs

Repo layout, full build/test workflow, conventions, and the contributor guide
live at **https://docs.pome.sh**.

## Before you build

- **npm only** — one root `package-lock.json` for `packages/*`; use
  `npm ci` / `npm install`.
- **Vocabulary: the product term is "task"** — "scenario" is retired (F-778,
  F-860, F-892). Never introduce "scenario" in new code, docs, or CLI copy.
  F-892 completed the CLI rename end-to-end: the command (`pome tasks`), scaffold
  (`./tasks/`), and the internal runner/schema surface (`src/task/`, `runTask*`,
  the `Task` type, `parseTask`, `taskSchema`). `pome scenarios` survives only as
  a hidden deprecated alias. The ONLY remaining sanctioned survivors are the
  serialized `scenario` / `scenario_*` keys that have a contract behind them —
  `meta.json`'s `scenario` slug (uploaded to cloud finalize; also read back by
  `pome eval` / `pome inspect` from run dirs older CLIs wrote) and the
  finalize / result / compile-seed wire fields (server contract; flip with
  W3/FDRS-653) — and the in-memory carriers whose value flows straight into
  them. F-933 renamed the two artifact keys that had no such contract:
  `runs/latest.json` now writes `task` (was `scenario`) and
  `runs/<task>/<session>/verdict.json` writes `task_path` (was
  `scenario_path`). F-1195 retired the verdict READ path's `scenario_path`
  tolerance: `verdict.json` is at artifact version 2, every file spelling it
  the old way is version 1, and the version check refuses those before the
  spelling could matter — so the normalize step was a dual-format reader that
  could no longer fire. Such a file is still RECOGNIZED as one of ours and
  `pome fix-prompt` names the skip (`staleVersionCount`) rather than dropping
  it silently. F-1411 gives the same treatment to a verdict.json that EXISTS
  on disk but cannot be read at all — truncated, hand-edited into an
  unexpected `state`, or valid JSON that is not one of our artifacts (the
  version check never even reaches such a file) — a different fact (nothing
  wrote this file correctly, vs. an older CLI that did) — counted and
  path-named separately as `unreadableCount` / `unreadablePaths`, never
  folded into `staleVersionCount`. A run dir with NO verdict.json is neither,
  and stays silently skipped.
- **The CLI (`cli/`) IS a root workspace member** — `workspaces: ["packages/*",
  "cli"]`, one `package-lock.json`, one `npm ci`. Use `npm run -w @pome-sh/cli
  ...` from the root. The former `cli/package-lock.json` and
  `cli/pnpm-workspace.yaml` (the changesets/manypkg root marker) are gone.
- **Internal `@pome-sh/*` deps are `"*"`** — sdk, wire and the five
  twins are workspace members resolved by npm's workspace linking, never from
  a registry. Never reintroduce an exact version pin between them: the exact
  pins drifted and installed a second registry copy of `@pome-sh/shared-types`
  (two zod schema identities at one runtime). This holds for `@pome-sh/wire`
  even though it is now published (F-949): in-repo consumers must keep
  resolving it through the workspace, never through GitHub Packages.

## Releases (`@pome-sh/cli`, `@pome-sh/adapter-claude-sdk`, `@pome-sh/checks`, `@pome-sh/wire`)

`@pome-sh/cli` and `@pome-sh/adapter-claude-sdk` are the ONLY things published
to npm for END USERS. `@pome-sh/checks` is also on npm but is not an end-user
package: it carries the five twins' check declarations, seed schemas and the
check DSL to `pome-sh/pome-cloud`, which grades every `[code]` criterion out of
them (F-1308). It re-exports rather than copies, and tsup inlines the twins'
compiled output, so it declares zero `@pome-sh/*` runtime dependencies — which is
also what keeps the `"*"`-only pin rule below intact for a PUBLISHED package. `@pome-sh/wire` is published to **GitHub Packages**
(`npm.pkg.github.com`) for internal cross-repo consumers — pome-cloud — while
ALSO still being bundled into both npm tarballs by tsup; publishing it was
additive and changed nothing about how this repo consumes it (F-949).
Everything else (`@pome-sh/sdk`, the five `@pome-sh/twin-*`) is a
`private: true` workspace member bundled into the two npm tarballs by tsup
(`noExternal: [/^@pome-sh\//]`).

**Do not write a version number in a PR.** Add an `## Unreleased (patch)` or
`## Unreleased (minor)` entry to that package's `CHANGELOG.md`; the number is
allocated on `main` after the merge by
[`.github/workflows/allocate-version.yml`](.github/workflows/allocate-version.yml)
(F-1511; reasoning and the rejected alternative in [`RELEASING.md`](RELEASING.md)),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pome-sh/digital-twins](https://github.com/pome-sh/digital-twins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
