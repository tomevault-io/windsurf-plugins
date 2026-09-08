---
trigger: always_on
description: Yarn 4 / Vite+ monorepo of everyday frontend DX instruments: SVG sprite pipeline, Figma integration,
---

# neodx

Yarn 4 / Vite+ monorepo of everyday frontend DX instruments: SVG sprite pipeline, Figma integration,
isomorphic logger, and a virtual file system, plus shared foundations and build tooling.

This file is the routing index: it keeps repo-wide constraints visible and points to the narrowest
source that owns the current decision.

## Session rules

These bind every session until the owner changes them.

**One branch.** Do the work on the standing branch `work` (cut over 2026-08-24 from `main` after
`strip-source-bridges` landed). Do not open PRs, merge to `main`, or add extra branches unless the
owner asks. `main` is landed history. Release automation on `main` is not everyday workflow.

**Changelogs, not publishes.** When a caller-visible change needs a changelog, add a Changeset with
the CLI and stop there:

```shell
yarn changeset add
```

Docs: [Changesets CLI](https://changesets.dev/guide/cli). This repo pins `@changesets/cli` **2.27.1**,
so `add` takes `--empty` and `--open`. It does not take the v3 `--patch` / `--minor` / `-m` flags.

**Publish freeze.** `strip-source-bridges` ([#180](https://github.com/secundant/neodx/issues/180)) is
on npm **1.1.1** and closed. Do not publish again until the owner names a **consolidated release**.
Queue later leftovers as Changesets on `work`. Do not run `yarn changeset version`,
`yarn changeset publish`, merge a **Version Packages** PR, recreate a closed Version Packages PR, or
re-run Release to push npm. The freeze exists so later work can share one release.

**Session close.** End every session with the [session-close gate](#session-close-gate). Name any
skipped check and why. Local green is not optional at close, even when CI is otherwise opt-in.

**Task names.** In issues, ledgers, and chat, lead with a readable slug, then the GitHub number:
`oxlint-typecheck` ([#179](https://github.com/secundant/neodx/issues/179)), not `R2-f`. Closed
program rows may keep old stream codes as history. Live work must not.

**Tighten what you touch.** Config, tools, and comments have gone vague in places (stale program
codes, duplicated tsconfig, dishonest overrides, dead paths, names that no longer match meaning).
Every session, after the assigned slice is green, make a bounded pass over files and tools this
session already opened: clarify ownership, delete dead config, collapse duplicates, and make names
match meaning. Do not start a repo-wide cleanup. Do not mix this pass with pack-contract or publish
work. Record leftover debt with a slug, not a silent skip. This rule outlives any one leftover.

## Package layers

| Layer               | Packages                                                                                                                                                                                                                                                  | Role                                     |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| Product (flagships) | `@neodx/svg`, `@neodx/figma`, `@neodx/log`, `@neodx/vfs`                                                                                                                                                                                                  | Caller-facing; ship docs + examples      |
| Foundation          | `@neodx/std`, `@neodx/colors`, `@neodx/fs`, `@neodx/glob`, `@neodx/pkg-misc`                                                                                                                                                                              | Shared helpers consumed by products      |
| Tooling             | `@neodx/autobuild` (private, `retire-autobuild` [#162](https://github.com/secundant/neodx/issues/162)), `@neodx/codegen` (private, `retire-codegen` [#163](https://github.com/secundant/neodx/issues/163)), `@neodx/scripts`, `@neodx/internal` (private) | Scaffold / quarantine / shared internals |
| Surfaces            | `apps/docs`, `apps/examples/**`, `apps/e2e/svg`                                                                                                                                                                                                           | VitePress docs, demos, visual e2e        |

- Dependencies flow foundation → product. Never import a product from a foundation.
- `@neodx/internal` is **build-time inline only**: a `devDependency` on `svg`/`vfs`/`figma`, never a
  published runtime `dependencies` entry, never a runtime import in `dist`. Enforced by
  `libs/svg/src/__tests__/internal-inline.test.ts`.
- The dependency graph must stay honest: run `yarn constraints` before claiming a change is done
  (`yarn constraints --fix` applies safe corrections).

## Command vocabulary (current)

Critical path is **Vite+** (`vp`). Yarn remains the package manager (`packageManager: yarn@4.3.1`;
`vp install` delegates to it).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [secundant/neodx](https://github.com/secundant/neodx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
