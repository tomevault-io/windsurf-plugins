---
trigger: always_on
description: Burger is a **hamburger menu with fullscreen navigation**: a stylesheet, one small
---

# AGENTS.md — the-burger

Burger is a **hamburger menu with fullscreen navigation**: a stylesheet, one small
ES module, and a demo page. Shipped as the npm package `the-burger` and a CDN
stylesheet. No framework, no runtime dependencies.

## Repo layout

```
src/
  burger.css       entry: declares @layer order + @imports each partial into a layer
  _tokens.css      burger.tokens  — the --burger-* theming API, including derived calc()
  _overlay.css     burger.overlay — the accent wash, scroll lock, box-sizing scope
  _menu.css        burger.menu    — the toggle button, its three bars, the brand
  _nav.css         burger.nav     — the nav list, links, slide keyframes
  scripts/burger.ts       TypeScript, ESM, auto-inits on import
  scripts/burger.test.ts  vitest + jsdom, JS contract only (see Testing)
  index.html       demo page (also the deployed site's source)
  img/             opengraph image
tests/
  burger.browser.spec.ts  Playwright, everything jsdom cannot see (see Testing)
dist/              BUILD OUTPUT, committed — css/, scripts/ (js + .d.ts), img/, index.html
site/              deploy artifact, gitignored, produced by build:site
```

`vitest.config.ts` picks up `src/**/*.test.ts` only; `playwright.config.ts` builds
and serves `site/` before running `tests/`.

## Commands

```bash
npm run build        # src/ -> dist/ (Lightning CSS + esbuild + tsc declarations)
npm run lint         # oxlint
npm run format       # oxfmt --write .
npm run check:types  # tsc --noEmit
npm run test:unit    # vitest run
npm run test:browser # playwright test (npx playwright install chromium once)
npm test             # lint + format + types + unit + build + publint + attw
npm run dev          # build, then serve site/ at :4321
```

esbuild compiles the TypeScript and `tsc -p tsconfig.build.json` emits only the
`.d.ts` that `exports.types` points at. Two tsconfigs on purpose: the root one is
`noEmit` and covers the tests, the build one excludes them.

## Testing

Two suites, split by what the environment can actually observe.

- `src/scripts/burger.test.ts` — **vitest + jsdom**, the JavaScript contract:
  toggling, Escape, focus return, closing on navigation, `inert`, teardown.
- `tests/burger.browser.spec.ts` — **Playwright + Chromium**, everything that comes
  out of a `:has()` selector, inherited `visibility`, a real scroll container, or a
  media query: viewport anchoring, the scroll container on short viewports, focus
  containment, touch targets, forced colors, reduced motion, and that no layout
  property is transitioned. jsdom implements none of this, so do not try to move
  these into the unit suite; they will pass vacuously.

`npm test` runs the unit suite. `npm run test:browser` runs the browser suite
(needs `npx playwright install chromium` once); CI runs both.

When emulating a media feature in the browser suite, use `page.emulateMedia()`
inside the test rather than `test.use({ reducedMotion })` — the latter lost to the
project-level `use` in `playwright.config.ts` and the assertion passed against
un-emulated defaults.

## Architecture conventions (enforced, not optional)

- **One piece of state**: `aria-expanded` on `.b-menu`. The stylesheet derives the
  overlay, scroll lock, bar rotation, and list animation from it with `:has()`.
  Never add a second `.open` class for CSS to key off; there is nothing to sync.
- **Cascade layers**: everything lives in
  `@layer burger.tokens, burger.overlay, burger.menu, burger.nav`. Layered rules
  lose to any unlayered author CSS, so consumers override Burger for free.
- **Tokens only**: never hardcode a colour or size, reference a `--burger-*`
  custom property from `_tokens.css`. Values computed from other tokens are
  themselves tokens (`--burger-nav-offset-block`), so overriding `--burger-size`
  recomputes every dependent value rather than half of them.
- **Logical properties**: `inset-block-start`, `padding-inline`, `inline-size`,
  `text-align: start` — never physical `left`/`right`/`top`/`bottom`/`width`.
- **Viewport-anchored, not document-anchored**: the wash, the toggle and the nav
  are all `position: fixed`. As `absolute` they pinned to the top of the document,
  so on a page taller than the window the menu opened off-screen. The demo is one
  viewport tall, which is why nothing caught it — test at 800x300.
- **The panel is the scroll container.** `.b-nav` is `inset: 0` with
  `overflow-y: auto` and `overscroll-behavior: contain`; the page behind stays
  `overflow: hidden`. Without it, a list taller than the window is unreachable —
  the wheel does nothing and focus cannot scroll a clipped link into view. Keep the
  list's padding on the `<ul>`, not `li:first-child`, so it is scrollable content
  rather than an offset the scroller knows nothing about, and keep the toggle
  `fixed` outside the scroller: a close button that scrolls away is a trap.
- **44px touch targets**: `--burger-target-size` is the floor for anything tapped.
  Links and the brand are `inline-flex` with `min-block-size`; as bare inline boxes
  they were 28px, the height of the text.
- **Gate hover behind `(hover: hover) and (pointer: fine)`**, or a tap leaves the
  link stuck in its hover state until you tap elsewhere.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mblode/burger](https://github.com/mblode/burger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
