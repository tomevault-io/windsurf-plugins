---
trigger: always_on
description: Operational knowledge for working in this repo. Design rationale is in
---

# Notes for coding agents

Operational knowledge for working in this repo. Design rationale is in
[DESIGN.md](DESIGN.md). User-facing docs live on the site
(`packages/site`, deployed to https://lazypromise.com); the root
[README.md](README.md) and `packages/core/README.md` only point there.
alien-signals docs stay on GitHub
([packages/alien-signals/.github/README.md](packages/alien-signals/.github/README.md));
`packages/alien-signals/README.md` is an NPM stub. `packages/site/README.md`
has the site's own operational notes.

## Layout

- pnpm workspace + turbo. `packages/core` is the library (`@lazy-promise/core`);
  `packages/interop` (`@lazy-promise/interop`) is a dependency-free leaf that
  core depends on: it owns `ErrorBox`/`NotAnErrorBox`/`UnboxError`/`Consumer`
  (re-exported by core) and the `*Like` types and guard for libraries that
  accept lazy promises without depending on core;
  `packages/alien-signals` is a proof-of-concept of async signals; at runtime it
  depends only on interop (core is a dev dependency for tests) and derives
  proxies via `original.constructor`;
  `packages/site` is the docs site (Astro, private, no `version` so
  `publish.sh` skips it); `packages/eslint-config` and
  `packages/typescript-config` are shared config.
- `temp/` is gitignored scratch space (probe scripts, patches).

## Commands

- Build: `npx turbo build:force` from the root. `build:force` emits even with
  type errors (`--noEmitOnError false || true`), so `build/types` is always
  refreshed. Turbo hashes package files, so source edits invalidate the cache;
  pass `--force` only after editing the shared config packages
  (`typescript-config`, `eslint-config`), which have no build task and so do
  not feed into dependents' hashes.
- Test everything: `npx turbo test`. Runs eslint, `tsc` (type tests), vitest,
  and a root prettier check (`prettier --list-different '**'`). Run
  `npx prettier --write <files>` on anything you edit or the check fails.
- Per package: `npx vitest run [file]`, `npx tsc` (noEmit), and
  `npx eslint . --max-warnings=0`.
- Coverage: tests execute `build/module`, not `src`, so run
  `npx vitest run --coverage --coverage.reporter=text --coverage.include='build/module/**'`
  in `packages/core` after a rebuild (coverage on `src/**` reports 0%).
  Coverage is 100% and must stay there; if a line ever has to be exempt,
  still run the report and check for regressions elsewhere. The one existing
  exemption is `disposeSymbol.ts`, excluded as a whole file in
  `vitest.config.mjs` because which branch runs depends on the Node version;
  v8 only supports file-level exclusion from config, and marker comments in
  the source would ship to clients.
- Benchmarks: `node scripts/bench.mjs [ref] --runs=5 --iterations=300000`
  compares the working tree against a git ref or npm version (default `HEAD`).
  The default iteration count is slow; run one benchmark process at a time.
- Publish: `.github/workflows/publish.yml` runs `scripts/publish.sh` per
  package, publishing when `package.json` version differs from npm and tagging
  `<name>@<version>`.

## Build/typecheck gotcha

- Test files import the package by name (`@lazy-promise/core`). That resolves
  via tsconfig `paths` to the package dir, then via `package.json` `types` to
  the compiled `build/types/*.d.ts`, not `src/`. After editing `src`, rebuild
  before type-checking or inspecting types in tests, or you will debug stale
  types.
- vitest transpiles with esbuild and does not type-check. `expectTypeOf` and
  `@ts-expect-error` tests only fail under `tsc`.
- Vite's SSR transform (vitest) snapshots imported bindings that are referenced
  in class field initializers into a `const` before the class, so a mutable
  export (`activeFrame` in `trace.ts`) read there is stale under vitest but
  fine in Node. Read such bindings in the constructor body instead.
- `build/` is gitignored; deleting it is always safe.
- Editor-only or CLI-only type errors are usually TS version skew (bundled VS
  Code TS vs the workspace one) or check-order dependent variance validation;
  see DESIGN.md.

## Conventions

- ESM sources; `tsc` emits `build/module` + `build/types`, babel emits CJS to
  `build/main`. ESLint config is `.eslintrc.cjs` per package.
- `LICENSE` is a committed copy in each package (npm pack does not include the
  root file and strips symlinks).
- Naming: `Sink` is the object passed to a producer (`sink.resolve/reject`);
  `Consumer` is the object passed to `.subscribe`; `Producer` is an object with
  `.produce`; `Job` is the disposable a producer returns (teardown);
  "subscription" is the disposable returned by `.subscribe()`. Classes follow
  `XxxConsumer`, `XxxJob`, and `XxxConsumerJob` when one object plays both
  roles.
- Style: early returns over `else`; no abbreviated names; minimal comments,
  especially on type-level code (the author prefers experimenting with types
  to reading prose about them).
- Docs (site) state invariants, not corner cases. The reader is not a computer:
  given the invariants they can infer the reasonable behavior in rare cases
  (that `sink.reject` after `sink.resolve` is ignored, how detached `finally`
  cleanup behaves) and experiment if they care. Such details belong in tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lazy-promise/lazy-promise](https://github.com/lazy-promise/lazy-promise) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
