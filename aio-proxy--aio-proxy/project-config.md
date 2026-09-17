---
trigger: always_on
description: aio-proxy routes model requests across configured upstream providers while keeping client-facing protocols stable.
---

# aio-proxy Agent Notes

aio-proxy routes model requests across configured upstream providers while keeping client-facing protocols stable.

<!-- CODEGRAPH_START -->

## CodeGraph

In repositories indexed by CodeGraph (a `.codegraph/` directory exists at the repo root), reach for it BEFORE grep/find or reading files when you need to understand or locate code:

- **MCP tool** (when available): `codegraph_explore` answers most code questions in one call — the relevant symbols' verbatim source plus the call paths between them, including dynamic-dispatch hops grep can't follow. Name a file or symbol in the query to read its current line-numbered source. If it's listed but deferred, load it by name via tool search.
- **Shell** (always works): `codegraph explore "<symbol names or question>"` prints the same output.

If there is no `.codegraph/` directory, skip CodeGraph entirely — indexing is the user's decision.
<!-- CODEGRAPH_END -->

## Repo Basics

- Bun workspace monorepo (`packages/*`) orchestrated by Turborepo.
- `packages/dashboard/AGENTS.md` is the authority for dashboard/frontend rules.
- Before considering a change complete, run `bun run preflight` (oxlint + oxfmt check + all unit tests), or at minimum `bun run check` plus the affected package's tests.
- Pull request titles follow commitlint / Conventional Commits (`type(scope): subject`). Do not put that prefix on changeset bodies.

## Changesets

Releases are driven by Changesets. All workspace packages share one lockstep version (`fixed` in `.changeset/config.json`), and only two packages get a published GitHub Release with notes: the `aio-proxy` CLI launcher and `@aio-proxy/plugin-sdk`.

- Every changeset that affects users MUST target a product package: `aio-proxy` (for CLI/proxy changes) and/or `@aio-proxy/plugin-sdk` (for SDK changes). This is what puts the note into a published Release.
- When the change actually lives in an internal package, also list that package alongside the product package, e.g. a `core` fix is a changeset targeting both `@aio-proxy/core` and `aio-proxy`.
- Never write a changeset that targets ONLY an internal or platform-binary package (`@aio-proxy/core`, `server`, `cli`, the plugins, `@aio-proxy/cli-*`). The `fixed` group still bumps `aio-proxy`, but its CHANGELOG entry would be empty, so `scripts/release.ts` skips its GitHub Release and the notes silently vanish.
- Keep the product package's bump level equal to the internal package's (internal `minor` -> `aio-proxy` `minor`).
- Use `bun changeset` to author them; commit the generated `.changeset/*.md` alongside the change. Do not run `changeset version`/`publish` by hand — CI owns both.
- A pending note describes the shipped state, not the state when it was written. When a change reverses, replaces, or drops something an earlier unreleased note announced, grep `.changeset/` for that behavior and correct or delete the stale note in the same commit. Notes are authored per task and released in one batch, so an unrevisited note ships as a Release describing a feature the code does not have.
- Keep a note short: one paragraph, at most 5 lines of body. It is a release note for users — say what changed for them, and for a fix what was wrong. Leave out implementation detail, file names, and the sequence of attempts. Do not prefix the body with an area label (`core:`, `cli:`, plugin short name); the frontmatter already lists the packages.
- Follow-up work on an unreleased change **rewrites** its existing note; it never appends a paragraph. A bug found and fixed before the feature ships never happened as far as the Release is concerned, so fold the corrected behavior into the original sentences and delete the intermediate story. A note that has grown past 5 lines is the signal to rewrite it.
- Write a second changeset only when the work is genuinely independent of the first, not to get under the line budget.

## Domain Language

Use these terms in code, docs, and discussion; avoid the listed synonyms.

- **Provider ID**: a stable identifier for an upstream provider. In user config, it is the key in the `providers` object. Avoid: provider name, provider key.
- **Provider priority**: an integer failover tier (`0..10000`, default `0`). Higher values are tried first. Avoid: order, rank.
- **Provider weight**: a finite authored number for same-priority traffic (default `1`, then `Math.round` and clamp to `0..10000`). Larger effective values receive more same-tier traffic. Avoid: order, rank.

## Coding Standards

### Utilities

- Search the codebase before adding a utility.
- Prefer `es-toolkit`, or a composition of its functions, for generic collection, object, string, and function utilities.
- Do not hand-write utilities without business meaning when `es-toolkit` already provides equivalent behavior.
- Keep trivial native JavaScript when it is clearer, such as `map`, `filter`, `some`, `every`, object spread, or a simple loop.
- Prefer narrow imports such as `es-toolkit/array`, `es-toolkit/object`, `es-toolkit/predicate`, and `es-toolkit/function`.
- Avoid `es-toolkit/compat` unless lodash-compatible behavior is explicitly required.
- Object shape checks:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aio-proxy/aio-proxy](https://github.com/aio-proxy/aio-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
