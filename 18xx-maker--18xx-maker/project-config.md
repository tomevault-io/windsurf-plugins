---
trigger: always_on
description: Prototyper for 18xx board games. The real product is SVG that people print at
---

# 18xx Maker

Prototyper for 18xx board games. The real product is SVG that people print at
exact physical sizes (tiles, maps, market charts, tokens, cards). Setup, scripts
and test details are in `DEVELOPMENT.md`.

## Testing

Commands (Node >= 24, pnpm; install the browser once with
`pnpm exec playwright install chromium --only-shell`):

```shell
pnpm test:run          # all vitest projects, once
pnpm test:run -u       # also update print output snapshots
CI=1 pnpm test:run     # reproduce CI (coverage floor, snapshots never written)
pnpm build && pnpm test:e2e   # Playwright against the built site (dist/site)
pnpm validate          # schema check of every src/data/**/*.json
```

### Full test run (what CI runs)

Run in this order before pushing; every step must pass:

```shell
pnpm install
pnpm exec playwright install chromium --only-shell   # once
pnpm pretty                                          # prettier check (pretty:fix to fix)
pnpm lint
pnpm validate
CI=1 pnpm test:run                                   # unit + component, coverage floor, no snapshot writes
pnpm build && pnpm test:e2e                          # Playwright on dist/site
pnpm build:app && pnpm build:sb                      # Electron and Storybook must compile
```

Also run `E2E_ELECTRON=1 pnpm test:export` (after `pnpm build && pnpm build:app`)
when touching `electron/`, the preload, render mode or `src/export`. This is
required before pushing, not optional: unit tests do not catch a changed export
(a PNG border changed image sizes that `e2e/export-files.js`, `e2e/cli.spec.js`
and `e2e/electron.spec.js` assert, and only CI noticed). An export change that
alters file sizes or contents updates those expected values in the same
commit. In the PR description, list which of these suites you ran. CI runs
the vitest suite in 3 shards on Linux (reports merged for the coverage check);
macOS and Windows run only the unit project on PRs.

Layout (projects are defined in `test.projects` in `vitest.config.js`):

- `unit` (node): `src/**/*.test.js`: logic in `src/util` and `src/state`. No
  jsdom: a test that reads `window` or `localStorage` imports
  `tests/support/windowStub.js` first (`resetWindow` resets it); one that
  needs a real DOM is a component test.
- `component` (real Chromium via `@vitest/browser` + Playwright):
  `tests/**/*.test.jsx` and `src/**/*.test.jsx`. Component tests do not use jsdom: it
  disagrees with Node on `Request`/`AbortSignal`, which react-router needs.
  Setup is `tests/support/setup.js`; render with `renderApp` from `tests/support/helpers.jsx`
  (real store + memory router built from `rootRoutes`).
- `e2e/*.spec.js`: Playwright on `vite preview` of `dist/site` (port 4318).
  Rebuild after any code change, specs never see the dev server.

Conventions:

- Print output is the contract. `tests/snapshots.test.jsx` snapshots every print
  page for a few games into `tests/__snapshots__/<game>/` (ids renumbered,
  decimals rounded to 3 places, `getBBox` stubbed, locale `en-US`). A React,
  Redux or UI change should change zero bytes there. Only run `-u` when an
  output change is intended, and review the snapshot diff. To cover another
  game, add its slug to `tests/snapshots.test.jsx`.
- `src/data/games/18Test.json` is the fixture game. Page roots expose
  `data-testid` hooks like `game-18Test-map`; add one when adding a page.
- **Always add an example of every new feature to `18Test.json`** (a new
  option, tile element, charter/market/token field, ...), so it renders in the
  print pages and the snapshots cover it. Extend an existing entry where one
  fits; add a new one only when the feature needs it.
- Every route in `rootRoutes` needs a smoke test entry (`tests/support/routes.js`,
  `tests/support/smoke.js`).
- CI enforces a 95% statement floor on `src/state/**`. The state layer is
  pinned by `src/state/*.test.js` (reducers, thunks, root state contract,
  persisted localStorage fixture); a change to stored shape needs a migration
  test.
- Test the rendered output, not the implementation. Chrome (nav, config drawer,
  home, docs) may change in a UI overhaul; print pages may not.
- Lefthook's pre-commit test step excludes `tests/**`; run `pnpm test:run`
  yourself before pushing.

## Visual changes

Snapshots and tests cannot see spacing, centering, size or stroke width, so
look at the output yourself before the first push. Two PRs took 8 and 2
follow-up commits on spacing and a missing border that a render would have
shown at once.

1. Build, then render the thing you changed zoomed in, with `pnpm shot`
   (`scripts/shot.mjs`, output in `shots/`, not committed):

   ```shell
   pnpm build
   pnpm shot 18Test tokens --clip 316,6,200,108      # a region, 4x
   pnpm shot 18Test tokens --sel "g:has(> circle)"   # one element
   pnpm shot 18Test tokens --vs /path/to/main/dist/site   # side by side
   ```

   `--vs` takes a build of `main` (build it once in another worktree) and
   writes `*-compare.png` with before and after. Open the PNG with the Read
   tool and look at it. A render you did not look at does not count.

2. Compare against the neighbours, not against your memory: put the new item
   next to an existing one in the same render (`18Test.json` has the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [18xx-maker/18xx-maker](https://github.com/18xx-maker/18xx-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
