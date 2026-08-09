---
trigger: always_on
description: Navio is a d3 widget for summarising and exploring tabular data: one column per
---

# Working on Navio

Navio is a d3 widget for summarising and exploring tabular data: one column per
attribute, one pixel row per record, with multi-level drill-down filtering. It
ships as a library (npm `navio`), so the public surface and the bundle shape
matter as much as the behaviour.

This file records the things that are **not** obvious from reading the code, and
the mistakes this repo has actually produced. Skim the "Landmines" section before
changing anything in `src/`.

## Commands

```bash
npm run check      # format:check + lint + unit tests + build. The gate.
npm run build      # rollup -> dist/{navio.js,navio.esm.js,navio.min.js}
npm test           # vitest, unit only
npx playwright test  # e2e. Needs a current dist/ - run build first.
npm run lint:fix   # eslint --fix
npm run format     # prettier --write
```

`npm run check` does **not** run the e2e suite. A change to `src/` is not
verified until both `npm run check` and `npx playwright test` pass.

**Check exit codes, do not grep output.** A rollup `SyntaxError` prints the word
"Error" capitalised and does not match a grep for `error`; that mistake pushed a
red build in this repo, and Playwright then silently tested a stale `dist/` from
the previous day. Capture to a file and test `$?`:

```bash
npm run build > /tmp/build.log 2>&1; echo "EXIT: $?"
```

## Layout

```
src/navio.js        ~2800 lines, ONE closure. Almost everything lives here.
src/filters.js      the five filter factories + filterFromValue (serialisation)
src/scales.js       scaleText, scaleOrdered, null-safe comparators
src/NavioWidget.js  reactivewidgets.org wrapper: .value, input events
src/index.js        UMD entry - DEFAULT EXPORT ONLY (see Landmines)
src/index.esm.js    ESM entry, named exports
build/ascii.js      escapes non-ASCII literals (replaces rollup-plugin-ascii)
build/verify-bundle.js  postbuild guard, runs automatically
docs/ai/            durable review docs - read FILTERING-MODEL.md before
                    touching filtering, sorting or selection
```

`src/navio.js` is a single closure. Its internal helpers are **not exported and
cannot be imported** — do not write a unit test that reaches into them, and do
not assume a function is available outside the file. Behaviour that lives in the
closure gets tested through the browser (see Testing).

`d3` and `popper.js` are **external** (see `rollup.config.js`). Navio does not
bundle them; it reads them off the host page. That coupling is invisible at load
and only fails on interaction — an example serving d3 v4 threw
`d3.pointer is not a function` only when someone sorted a column. A unit test
(`test/unit/examples-d3-version.test.js`) now fails if any example pins d3 < 7.

## Conventions

**Prefer standard browser controls.** When the platform already has an element
for the job, use it rather than building the behaviour out of divs and click
handlers: `<details>`/`<summary>` for a collapsible section, `<dialog>` for the
settings panel, `overflow-y: auto` for a scrolling list, a real `<button>`,
`<select>` or checkbox for a control. The native element brings its keyboard
behaviour, its ARIA state, focus handling and find-in-page for free, and a
hand-rolled version is a worse copy that has to be maintained. This is a
maintainer preference, stated directly — apply it to anything new in the
settings panel.

The exception is when the native behaviour is measurably wrong for the case, and
"measurably" means a number: `dialog.showModal()` was tried for the settings
panel and removed, because it centres in the VIEWPORT — with two Navios on a
page the panel appeared nowhere near the widget it belonged to — and its one
real advantage, the top layer, cannot escape the sandboxed cross-origin iframe
an Observable notebook renders its cells in.

## Landmines

Each of these has cost real debugging time here.

**Filters are evaluated once, at creation.** They are not live predicates
re-run on redraw. `applyFilters` materialises `selectedFlags`; re-sorting a level
deliberately never calls it, so sorting cannot change the selection. Range
filters compare *visual positions*, not values. Read
`docs/ai/FILTERING-MODEL.md` — a plausible mental model here is wrong and
produces designs that look right and are not.

**`nv.nestedFilters = false` is a second, barely-walked code path.** With it on
(the default) `applyFiltersAndUpdate` grows `dataIs` a level per filter, so the
level chain is always longer than 1 by the time anything downstream runs. With
it off the chain stays at length 1, and code guarded on `dataIs.length <= 1`
fires for the first time — that is how `deleteSubsequentLevels`, whose early
return forgot to hand `_dataIs` back, made brushing throw "Cannot read
properties of undefined (reading 'length')". The setting has a checkbox in the
settings panel, so users reach it. Any function that takes `_dataIs` and is
assigned to its caller's variable must return it on **every** path.

**Hiding every column empties the attribute scale's domain.** `xScale.domain()`
becomes `[]`, `domain()[0]` and `domain()[length - 1]` are `undefined`,
`scaleBand` answers `undefined` for a value it does not know, and
`levelScale(level) + undefined` is `NaN`. d3 then writes that into SVG

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [john-guerra/navio](https://github.com/john-guerra/navio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
