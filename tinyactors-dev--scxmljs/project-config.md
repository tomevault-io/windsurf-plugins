---
trigger: always_on
description: <!-- Source of truth for agent guidance.
---

# scxmljs

<!-- Source of truth for agent guidance.
     Read by Amp as AGENTS.md and by Claude Code via the CLAUDE.md symlink. -->

## Overview

`@tinyactors/scxmljs`: a correctness-first SCXML 1.0 interpreter (ECMAScript data model)
that runs on DOM elements, plus custom elements that render running statecharts.
The package is published on npm (0.1.0); `PUBLISHING.md` is the release checklist and
records the decisions (custom elements only, no outside contributions, CI = one script).

Layout:

- `packages/scxmljs/`: the library (`src/`, `test/`). Two entry points with the same API:
  `index.ts` (sandboxed data model, QuickJS/WebAssembly) and `trusted.ts` (host JS engine).
  `explorer.ts` is the `./explorer` entry (`src/explorer/`: element, view-model, styles);
  `view.ts` is the `./view` entry (`src/view/`: `<scxml-view>`, its layered layout, strings,
  styles); it loads data model engines only through `import()`. `src/ui/` holds what the
  elements share: `theme.ts`, the `--scxml-*` token contract and neutral theme, and `announcer.ts`;
  `src/themes/*.css` are optional theme stylesheets. The explorer must import internal modules
  only (never `index.ts`), so it never pulls in QuickJS.
- `examples/playground/`: private demo app (Bun server, the explorer with sample systems,
  the library `<scxml-view>` on `/element`, a GitHub-webhook gatekeeper).
  Its tests live in `examples/playground/test/`.
- `examples/llm-chat/`: the multi-client LLM chat demo (`/demos/llm-chat/`): host and client charts,
  scenario charts, `PROTOCOL.md` (every processor/invoker's messages; keep `src/protocol.ts` in step),
  a simulated model and tools on one clock, Claude via the SDK with the visitor's key (loaded lazily).
  `site/client/llm-chat.ts` renders it. Tests: `examples/llm-chat/test/` (headless) and `tests/site/`.
- `examples/pi-durable/`: Earendil's Pi Durable as statecharts (`/demos/pi-durable/`): one chart per task
  kind (`pi.generation`, `pi.tool`, `pi.compaction`, `shop.checkout`, `shop.payment`, `app.reminder`), `harness.scxml`,
  `client.scxml`; storage survives "Kill process", a new process resumes every task from its checkpoint. The tour
  (`charts/tour/`, one chapter per section of the post; `src/post.ts` holds the verbatim quotes the ¶ popovers show)
  drives it through the `stage` processor. `PROTOCOL.md` lists every commit, invoker and event: keep it in step with
  `src/harness.ts`. `site/client/pi-durable.ts` renders it. Tests: `examples/pi-durable/test/` (every chapter runs to
  its end) and `tests/site/`.
- `conformance/`: W3C SCXML IRP suite. `fetch.ts` downloads and converts it, `run.ts` runs it.
- `tests/browser/`: Playwright tests (Chromium, Firefox, WebKit) against `server.ts`, which serves
  the playground pages and `fixtures/*.html` (the BUILT package via an import map; `?csp=` adds a
  Content-Security-Policy). Specs tagged `@visual` are screenshot comparisons that only run inside
  the pinned Playwright Docker image; baselines live in `specs/__screenshots__/`.
- `site/`: the website https://scxmljs.tinyactors.dev (GitHub Pages). `scripts/site/build.ts` renders
  it into `_site/` (gitignored): `site/src/layout.ts` is the page shell (head, nav, footer),
  `site/src/pages.ts` the hand-written pages, `site/src/docs.ts` renders `docs/*.md`, `SECURITY.md`
  and `CHANGELOG.md` (sidebar `GROUPS`: a new guide must be added there, or the build fails),
  `site/client/*.ts` are the browser entries (bundled from source with hashed names),
  `site/styles/site.css` the stylesheet (playground design tokens + the Tinyactors theme),
  `site/charts/` extra charts, `site/public/` static files (`og.png` from `mise run site:og`).
  `/playground/` is the live editor: `site/client/playground.ts` (+ `playground-editor.ts`,
  CodeMirror, lazy), `diagnostics.ts`, `share.ts`; examples in `site/src/playground-examples.ts`.
  User charts run only in the sandboxed engine and load only the playground's own `/charts/` files.
  Smoke tests: `tests/site/` (`*.pw.ts` Playwright, `*.test.ts` bun).
- `examples/frameworks/{react,vue,svelte,angular}/`: real apps, each its own project and
  lockfile, depending on the library via `file:` (so `dist/` must be built). Their component
  files ARE the snippets in `docs/frameworks.md` (`<!-- doctest: app file=… -->` checks they're
  identical): edit both together. Biome and the root typecheck skip them; each app's toolchain
  checks it.

## Conventions

- Bun for everything: `bun test`, `bun run`, `Bun.serve`, `bun build`. Bun workspaces.
- mise is the task runner. Tasks live in `mise.toml` (committed). Pitchfork runs the playground server.
- Correctness first: any change to `packages/scxmljs/src` must keep the conformance suite at
  160/160 mandatory tests in **both** data models.
- Tests use `VirtualClock` and happy-dom's `DOMParser`; never rely on real time.
- Inside the repo, `@tinyactors/scxmljs[/*]` resolves to `packages/scxmljs/src` through the root
  `tsconfig.json` `paths` (Bun and tsc honour it), so no build is needed to develop. The published
  package resolves to `dist/` through `exports`.
- Don't commit, publish or trigger CI unless asked.

## Commands


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tinyactors-dev/scxmljs](https://github.com/tinyactors-dev/scxmljs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
