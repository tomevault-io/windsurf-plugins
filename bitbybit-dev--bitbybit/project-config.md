---
trigger: always_on
description: generates differs from what is committed. Never edit a generated file - change the kernel, or the
---

# CLAUDE.md - the bitbybit open-source monorepo

MIT-licensed, and the source of truth for the CAD algorithms and the published `@bitbybit-dev/*` npm
packages. It is consumed as a git submodule elsewhere, but it stands alone: everything here builds and
tests without it. Start with `README.md` for the overview, `CONTRIBUTING.md` before opening a PR.

## Layout

| Directory | What it is |
|---|---|
| `packages/dev/*` | the 14 published npm packages - see `packages/dev/CLAUDE.md` |
| `docs/` | the Docusaurus site for learn.bitbybit.dev, including the generated API reference |
| `examples/` | runnable examples per framework (angular, nextjs, nuxt, node, vite, react); `examples/scripts/examples.mjs` installs, builds and audits each one weekly from the registry, in examples.yml, and `examples/scripts/local.mjs` runs them against this repository's own packages instead |
| `languages/` | the API help text per locale, keyed by each member's dotted path, and `doc-paths.json`, the map from documentation page names to those keys (`API_DOCS_GUIDE.md`) |

## Building the packages

The packages form a dependency DAG, and the order has one source: each package's `package.json`
dependencies. `scripts/gen-ts-references.mjs` turns them into TypeScript project references -
every `tsconfig.bitbybit.json` is a composite project that references the siblings its manifest
declares, and `tsconfig.build.json` at the root references all eleven - so `tsc -b` orders the
compiles itself and rebuilds only what changed. `npm run build-packages` is `pnpm -r run build-p`: pnpm orders the eleven stagings by the same
manifests, each `build-p` compiles with `tsc -b` and stages dist/ for publishing.
`npm run rebuild-all-packages` empties every dist first; `tsc -b tsconfig.build.json --verbose`
prints the order it derives and what it considered up to date.

All three of a package's TypeScript configs are generated: the build config, the strict view, and the
`tsconfig.json` an editor and a lint run pick up - the build config's base and sibling paths without
its emit settings or exclusions, so the tests and mocks are in the project there. After changing a
dependency between packages, run `npm run gen:references` and commit the result; `check:references`,
the first step of `npm test`, fails when any of the three is out of date.
Three things are placed on purpose: the build info sits in each dist/, because `tsc -b` trusts it
over the outputs and it has to vanish with the dist it describes (`copy-package`'s `.npmignore` keeps
it out of the tarball); every build config excludes `dist` and `coverage`, which TypeScript would
otherwise read as inputs; and every config states its `outDir`, which is how TypeScript keeps that
directory out of the project - a stale one reads the built `dist` back in as source, silently.

## The workspace

The fourteen packages under `packages/dev/` are one pnpm workspace (`pnpm-workspace.yaml`): one
`pnpm install` at the root - `npm run ci-packages` is exactly that, frozen to the lockfile - installs
all of them, and a sibling dependency whose exact pin matches the sibling's version becomes a symlink
instead of a registry copy (`linkWorkspacePackages`). One `pnpm-lock.yaml` replaces the per-package
npm locks; `npm run refresh-lockfile` rewrites it without touching node_modules. The manifests keep
exact registry pins on purpose and never the `workspace:` protocol: `dist/` is what npm publishes, and
`copy-package` derives its manifest through `scripts/dist-manifest.mjs`, which refuses a `workspace:`,
`link:` or `file:` specifier. A dependency's install script runs only when `allowBuilds` lists it, so
a new native dependency shows up as a decision, not a silent skip. Node comes from `.tool-versions`
and pnpm from `packageManager`, which pnpm switches to on its own.

Every dist-published manifest also carries an `exports` map derived from its tree by `npm run
gen:exports` (the root, every directory index under `lib/`, every kernel module, then patterns), with
the `@bitbybit-dev/source` condition first in each entry: a consumer that declares the condition
resolves the TypeScript sources - the shared test configuration does, through `resolve.conditions`,
so a suite sees a sibling's edit without a rebuild - and one that does not resolves `dist/`. The map
never reaches npm: `dist-manifest.mjs` drops it with `devDependencies` and `scripts`, and a published
package resolves through `main` and `types` as every version has, because an exports map in a tarball
would refuse the extensionless deep imports the examples make. `npm run check:exports` holds every
manifest to that shape and every built `dist/package.json` to the derivation.

pnpm's layout is strict: a package resolves only what its own manifest declares, where npm's flat
hoisting let it reach anything a sibling had installed. Every import in `lib/` must therefore be a
dependency of that package - the engine packages import `@bitbybit-dev/base`, the three workers,
`jsonpath-plus` and `verb-nurbs-web` directly, and declare them. Verify a build from a clone outside
your home directory: a stray `~/node_modules` above the checkout satisfies an undeclared import on
your machine and nowhere else, which is how one reached CI.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitbybit-dev/bitbybit](https://github.com/bitbybit-dev/bitbybit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
