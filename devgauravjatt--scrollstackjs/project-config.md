---
trigger: always_on
description: Working notes for coding agents in this repo. This file is the deep reference:
---

# AGENTS.md

Working notes for coding agents in this repo. This file is the deep reference:
invariants, gotchas, and the per-file source map. The human-facing docs are
[`README.md`](./README.md) (usage), [`CONTRIBUTING.md`](./CONTRIBUTING.md) (the short
path to landing a change), [`DECISIONS.md`](./DECISIONS.md) (why the architecture is
the way it is), and [`STATUS.md`](./STATUS.md) (built vs. roadmap).

**Read `DECISIONS.md` before changing engine behavior.** Most "obvious improvements"
already have an ADR explaining why they were rejected.

## Commands

```bash
pnpm install
pnpm run build       # tsc per package, topological (core before adapters)
pnpm test            # 154 tests / 22 files across the six packages
pnpm run typecheck   # tsc --noEmit per package
pnpm run verify      # build + typecheck + test — run this before declaring done
pnpm run lint        # oxlint over packages/ + examples/ — type-aware
pnpm run lint:docs   # docs/ separately (needs `cd docs && pnpm install` first)
pnpm run format      # oxfmt --write; `format:check` for CI
pnpm run check       # lint + format:check
```

**Linting is type-aware** (`options.typeAware` in the root `.oxlintrc.json`, powered
by the `oxlint-tsgolint` devDependency), so it needs the same things `typecheck`
does: `packages/core/dist` must exist before the adapters lint clean. `typeAware`
is honoured **only in the root config** — it can't be scoped per package — so it
applies to whatever paths a run is pointed at. That's why `lint` targets
`packages examples` (always provisioned by the root install) and `docs` has its own
script: type-aware linting can't resolve types there until `docs/` is installed.

Single package: `pnpm --filter @scrollstackjs/core test`.
Single test file: `pnpm --filter @scrollstackjs/core exec vitest run tests/retry.test.ts`.

**Adapters compile against `packages/core/dist`, not its source.** After editing
core, run `pnpm --filter @scrollstackjs/core build` before typechecking or testing an
adapter, or you will chase phantom type errors.

**Examples are not covered by `pnpm run typecheck`** — they only define a `dev`
script. Check one explicitly:

```bash
pnpm --filter @scrollstack-example/react-live-demo exec tsc -p tsconfig.json --noEmit
pnpm --filter @scrollstack-example/react-live-demo dev   # needs packages built first
```

**`docs/` is a separate pnpm project**, not a workspace member — it has its own
`pnpm-workspace.yaml` so a VitePress upgrade can't perturb the library build.
Root `pnpm install` does not touch it:

```bash
cd docs && pnpm install && pnpm run build   # fails the build on dead links
```

**There is no CI workflow running the test suite on pull requests.** `pnpm run verify`
on your machine is the only gate. The one workflow that exists deploys the docs.

## Docs deployment

The site deploys to <https://scrollstack.js.org> via `.github/workflows/docs.yml`
(GitHub Pages, Actions-based publishing). Two things there are load-bearing:

- **Build order.** Root `pnpm run build` runs first, because docs resolves
  `@scrollstackjs/{core,vue}` through `link:../packages/*` → `dist/`.
- **`base` stays `'/'`.** The custom domain serves the site at the root. VitePress
  bakes `base` into every asset URL at build time and has no relative-base mode, so
  building with `DOCS_BASE=/scrollstackjs/` and serving from the apex domain 404s
  every stylesheet, script, and font while `index.html` itself still returns 200.
  That env var exists only for building the bare `…github.io/scrollstackjs/` URL.

`docs/docs/public/CNAME` is js.org's proof of ownership. Under Actions publishing
GitHub reads the custom domain from repo settings, so don't rely on that file alone
if the domain ever needs re-attaching.

## Layout

```
packages/
  core/     @scrollstackjs/core      engine, state machine, retry, observer contract
  react/    @scrollstackjs/react     useInfiniteScroll (useSyncExternalStore)
  vue/      @scrollstackjs/vue       useInfiniteScroll (shallowRef)
  svelte/   @scrollstackjs/svelte    createInfiniteScroll (returns a store)
            each adapter also has src/virtual.ts -> the `/virtual` entry point
  virtual/  @scrollstackjs/virtual   virtualizer: layout.ts (pure) + virtualizer.ts
                                     (side effects) + scroller.ts (element vs window)
  devtools/ @scrollstackjs/devtools  dev-only panel: store.ts (logic) + panel.ts (DOM)
examples/  {react,vue,svelte}-live-demo  — all 7 features, Tailwind v4, real APIs
           the three live demos mirror docs/demo; change one, change all three
           react-live-demo-with-devtool  — a copy of the React demo with the
           devtools panel on FeedDemo; a demo change means a fourth edit here
docs/      VitePress site — docs/docs/{guide,api}/*.md, config in .vitepress/
           .vitepress/theme/demo/*.vue are live demos built on @scrollstackjs/vue
```

The docs site links `@scrollstackjs/{core,vue}` from `packages/*/dist`, so **build
the library before building the docs** or the demo imports fail. The demos call
public APIs (Rick and Morty, PokéAPI, JSONPlaceholder) at runtime only — the
build itself needs no network.

Core source map: `engine.ts` (orchestration + all side effects) · `state.ts` (pure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devgauravjatt/scrollstackjs](https://github.com/devgauravjatt/scrollstackjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
