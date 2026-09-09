---
trigger: always_on
description: Guidance for AI coding agents working on this repository. For human onboarding
---

# AGENTS.md

Guidance for AI coding agents working on this repository. For human onboarding
(forking, PR etiquette), see [CONTRIBUTING.md](./CONTRIBUTING.md).

## Repository layout

npm workspaces monorepo, ESM (`"type": "module"`). For the supported Node
range see [CI](#ci) below — it is stated once, there.

- `packages/typedoc-plugin-markdown` — the core plugin. Most work happens here.
- `packages/*` — utility plugins (`typedoc-plugin-frontmatter`, `typedoc-plugin-remark`)
  and framework themes (`typedoc-vitepress-theme`, `typedoc-docusaurus-theme`,
  `docusaurus-plugin-typedoc`, `typedoc-github-wiki-theme`, `typedoc-gitlab-wiki-theme`).
  All depend on the core plugin.
- `devtools/` — internal build tooling (`@devtools/*`, not published). In particular
  `devtools/packages/prebuild-options` generates code and docs from option declarations.
- `docs/` — the public docs site (https://typedoc-plugin-markdown.org), Nextra 4 / Next.js.
  It is **not** an npm workspace; it has its own `package.json` and lockfile.

In the core package, `@plugin/*` is a tsconfig path alias for
`packages/typedoc-plugin-markdown/src/*` (resolved at build time by `tsc-alias`).

## Generated files — do not edit directly

Editing any of these typechecks fine and then your change is silently erased.
There are two generators, with very different triggers.

**Overwritten by the `prebuild` step of every build:**

| Generated file (core package) | Source of truth |
| --- | --- |
| `src/types/options.ts` | `src/options/declarations.ts` (JSDoc + declaration shape) |
| `src/theme/context/resources.ts` | files in `src/theme/context/{templates,partials,helpers}/` |
| `docs/content/docs/options/*.mdx` (and per-package option docs) | each package's `src/options/declarations.ts` |

**Overwritten by `npm run docs` at the repo root:**

| Generated file | Source of truth |
| --- | --- |
| `README.md` (repo root) | the `Packages` table is built from each package's `description`; surrounding prose is inline in `devtools/scripts/generate-readmes.ts` |
| `packages/*/README.md` | `## Overview` and `## Features` of that package's `docs/content/**/index.mdx`, plus the install snippet in `PACKAGE_README_CONTENT` |
| the table of contents in `CONTRIBUTING.md` | its own headings, via `remark-toc` |

`npm run docs` is **not** wired into any build, test or CI job — nothing runs
it for you and nothing fails when its output is stale. Run it by hand after
changing a package `description` or the `## Overview` / `## Features` sections
of a docs index page, and commit the result alongside the change.

Note `packages/*/README.md` is the package's npm listing page. It is read by
users, but it still does not warrant a changeset — see the Changesets section
below.

**To add or change an option:** edit `src/options/declarations.ts` (the JSDoc
comment becomes the public documentation), then run `npm run build` in that
package — prebuild regenerates the types and docs. Same model applies in the
other packages that declare options.

**To add a theme partial/template/helper:** create the file in the appropriate
`src/theme/context/` subfolder; `resources.ts` is regenerated from the folder
contents on build.

## Build and test

Build order matters: the themes and utility plugins build/test against the core
plugin's `dist`, so build core first (workspace order already handles this):

```bash
npm run build-all        # build every workspace package
npm run test-all         # test every workspace package
```

Scoped to one package (preferred while iterating):

```bash
npm run build --workspace typedoc-plugin-markdown
npm run test --workspace typedoc-plugin-markdown
```

### How the core tests work (fixtures + snapshots)

The pipeline, in order:

1. `pretest` wipes `test/fixtures/out` and runs **TypeDoc itself** over ~29
   fixture configs (`test/fixtures/configs/*.cjs`), each rendering the sample
   projects in `test/fixtures/src` to markdown in `test/fixtures/out`. This is
   the slow step.
2. Mocha specs (`test/specs/*.spec.ts`, run via tsx) read those output files
   and compare them against `test/__snapshots__/*.snap`.

Two consequences that are easy to get wrong:

- **Fixtures run the plugin from `dist`, and `pretest` does not rebuild it.**
  After editing `src`, run `npm run build` before `npm run test`, or the
  fixtures — and therefore the tests — exercise your *pre-edit* code.
  (`npm run build-and-run` = build + fast dev fixture subset.)
- **A missing snapshot passes silently**: the harness writes the `.snap` on
  first sight and asserts against what it just wrote. New output files are
  therefore *not* covered until their snapshot is committed and reviewed, and
  deleting a `.snap` file "fixes" a failure without fixing anything. Treat new
  or deleted `.snap` files in a diff with the same suspicion as changed ones.

Fast iteration: after one `build` + `npm run fixtures`, run specs directly
with `npm run mocha` (add a spec path to run a single file) — no need to pay
for fixture regeneration when only spec/snapshot files changed. Rebuild
fixtures whenever `src` or `test/fixtures` changes.

### Snapshot failures

1. **Read the diff first.** A failing snapshot is usually the test doing its job.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [typedoc2md/typedoc-plugin-markdown](https://github.com/typedoc2md/typedoc-plugin-markdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
