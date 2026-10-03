---
trigger: always_on
description: Instructions for AI coding agents (and humans) working on LichessDotCom. Read
---

# AGENTS.md

Instructions for AI coding agents (and humans) working on LichessDotCom. Read
this file whole before changing anything. The code's design is detailed in
[docs/architecture.md](docs/architecture.md), the Game Review's in
[src/page/review/README.md](src/page/review/README.md).

## The project in brief

- A browser extension (Manifest V3, Chrome and Firefox) that gives
  lichess.org a modern look and feel, plus a Game Review with a coach.
- TypeScript 7, bundled by rolldown (`scripts/build.ts`) into
  `dist/<target>/`. The browser loads the build, never `src/`.
- Two scripts run in each Lichess tab, in two JavaScript worlds that share
  only the DOM and `window.postMessage`: `src/content` (isolated world: the
  extension's APIs and files) and `src/page` (page world: Lichess's own
  objects). `src/shared` serves both and never touches `chrome.*`.
  `src/background` is the service worker. `src/styles` is joined into
  `content.css`.

## The quality bar

This codebase is held to professional standards: every change must read as
if a senior engineer wrote it for code review. It was rewritten once already
because a contributor called it AI-generated and unclean. Don't let it slide
back. `pnpm check` enforces much of what follows; the rest is on you.

**Types**

- Strict TypeScript. Never `any`, never a type assertion (`as`, `<T>x`, `!`;
  `as const` is fine), never `@ts-ignore` or a lint-disable comment.
- Every piece of data from outside (JSON, fetch responses, `postMessage`,
  storage, `#page-init-data`, Lichess's globals) is validated with a
  zod/mini schema (`import { z } from 'zod/mini'`), and its type is
  `z.infer` of that schema, not written by hand.
- Lichess's live objects keep their identity: narrow them with
  `createGuard(schema)` (`#shared/guards.ts`), not a parse, which copies.
- DOM lookups narrow with `instanceof` through `queryOne`, `queryAll` and
  `closestTo` (`#shared/dom.ts`).

**Structure**

- One concern per module. Pure logic (parsing, geometry, text, rules) lives
  apart from the code that touches the DOM, so it can be tested alone.
- A feature is a folder under `src/content/` or `src/page/` whose module
  exports a `Feature`, listed in the world's `index.ts` in start order.
  Content features that follow the page register with `onEveryTick`
  (`#content/sync-loop.ts`).
- Code used by more than one feature goes in `src/shared/`; look there
  before writing a helper. Test-only helpers go in `src/shared/testing/`.
- Imports across folders use the `#` aliases of `package.json`'s `imports`
  (`#shared/…`, `#content/…`, `#page/…`, `#background/…`, `#scripts/…`,
  `#manifest`), never `../`. `./` is for the same folder or a folder below
  it. Keep the `.ts` extension.
- At most 250 lines of code per file and 60 per function, 4 parameters (use
  an options object), complexity 15, no nested ternaries. Split by what the
  code does, not to dodge the limit.

**Names and comments**

- Names say what things are: `square`, `whiteShare`, `moveTimes`, never
  `sq`, `ws`, `mt`. One letter only for loop indexes (`i`, `j`, `k`),
  coordinates (`x`, `y`) and type parameters.
- Comments are short and plain, and say why: a Lichess quirk, a browser
  pitfall, a constraint that isn't visible in the code. One or two lines is
  the norm. Never narrate what the code does, never write essays, and never
  use an odd, flowery or "AI" voice. A module may open with a few lines on
  what it's for.

**DOM, performance and security**

- Never move or remove Lichess's DOM nodes (snabbdom breaks). Add our own,
  prefixed `cdc`, and rearrange with CSS.
- The content script runs on every page and its tasks every 250 ms: a task
  must cost next to nothing when nothing changed. Write only what changes
  (`setData`, `setStyleProperty`, `classList.toggle(name, force)`), and
  avoid layout reads in hot paths. No endless `requestAnimationFrame` or
  animation loops, no `:has()` in hot places (see the pitfalls).
- Markup built as text goes through the escaping `html` template
  (`#shared/html.ts`); `trustedHtml` is for constants only. A message
  handler checks `event.source === window` and validates with a schema
  (`#shared/protocol.ts` does both).
- Everything of ours is prefixed `cdc`: classes, ids, data attributes, CSS
  variables, storage keys (listed in `StorageKey` / `SessionKey`).

**Tests**

- Every change ships with tests, written once the user has validated it
  (see [How a change goes](#how-a-change-goes)). Unit tests sit next to the
  code (`name.test.ts`, vitest in happy-dom): logic through its exports, DOM
  features by building the markup Lichess serves and checking what we add.
  User-visible flows get an end-to-end test in `tests/e2e`.
- See each new test fail once before trusting it: a test that can't fail is
  not a test. Assert on real outputs, not on a mock's own answers.
- `fixtures/legacy*.json` are outputs recorded from the original code. They
  are the reference for behavior: never regenerate them from the new code.
  If a change must alter one, it's a deliberate behavior change: say so in
  the commit. Compare their fractions through `nearly`
  (`#shared/testing/numbers.ts`): `Math.exp` and `Math.log` differ in the
  last bit between macOS and Linux.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theophile-wallez/LichessReimagined](https://github.com/theophile-wallez/LichessReimagined) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
