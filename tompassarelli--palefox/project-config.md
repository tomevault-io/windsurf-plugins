---
trigger: always_on
description: This file is loaded into Claude's context for every conversation in this repo.
---

# CLAUDE.md — palefox project guide

This file is loaded into Claude's context for every conversation in this repo.
Keep it short, current, and oriented toward "what would help me NOT make the
same mistake twice."

---

## What palefox is

A chrome-privileged userscript bundle that runs inside Firefox via
[fx-autoconfig](https://github.com/MrOtherGuy/fx-autoconfig). It rewrites the
sidebar to host a tree-tab panel with vim keybindings, plus drawer-level chrome
restructuring (compact mode, urlbar relocation, etc.).

Scripts run in the browser's privileged scope — they touch `gBrowser`,
`Services`, `ChromeUtils`, raw XUL elements, etc. They do NOT run in a normal
web page sandbox.

## Repo layout

```
src/tabs/        palefox-tabs.uc.js — sidebar tree-tab panel
  index.ts         orchestrator (wires factories, owns init)
  vim.ts           keymap, ex-mode, picker, search, blacklist
  rows.ts          row DOM creation / sync / visibility (factory)
  drag.ts          drag-and-drop (factory)
  layout.ts        panel positioning (factory)
  menu.ts          context menu (factory)
  events.ts        TabOpen/Close/Move/Select wiring
  history.ts      SQLite temporal substrate (events, sessions, retention)
  snapshot.ts      tree → envelope helpers + makeSaver
  content-focus.ts cross-process editable-element detection bridge
  helpers.ts       pure tree walks (treeData, levelOf, subtreeRows, …)
  state.ts         shared mutable singletons + WeakMaps
  types.ts         Tab, TreeData, Group, Row, SavedNode
  constants.ts     INDENT, CHORD_TIMEOUT, etc.
  log.ts           createLogger() — pfx.debug-gated
  *.test.ts        Tier 1 unit tests (bun test)

src/drawer/      palefox-drawer.uc.js — chrome restructuring
  index.ts         thin orchestrator — wires factories, runs the Ctrl+L keymap
  layout.ts        toolbox/urlbar reparenting + width sync + width pref (factory)
  drag-overlay.ts  -moz-window-dragging overlay over empty sidebar space (factory)
  banner.ts        HTTP not-secure warning banner (factory)
  compact.ts       compact-mode state machine (factory)
  urlbar.ts        floating urlbar + Ctrl+J/K suggestion nav (factory)
  sidebar-button.ts  custom #pfx-sidebar-button + context menu (factory)

src/firefox/     Firefox adapter layer — typed wrappers around chrome globals
  tabs.ts          gBrowser tab ops (allTabs / selectedTab / pinTab / …)
  prefs.ts         Services.prefs get/set/observe
  observers.ts     Services.obs add/remove/notify
  files.ts         IOUtils + PathUtils + profile-relative helpers
  dom.ts           createXULElement factories typed by tag
  window.ts        well-known chrome IDs + system principal + ESM loader

src/platform/    Palefox semantic platform layer (the capability API)
  index.ts         exports `Palefox` namespace; one per chrome window
  scheduler.ts     central scheduler with markDirty/flush/diag
  tabs-reconciler.ts  bridges Firefox tab events into the scheduler
  window.ts        PalefoxWindow facade (`Palefox.windows.current()`)
  window-tabs.ts   `WindowTabsAPI` — list/selected/pin/close/etc.

src/types/chrome.d.ts   ambient chrome globals + DOM augmentation
src/hello/              smoke-test stub

chrome/
  JS/*.uc.js              BUILT — do NOT hand-edit
  palefox.css             core theme + aggregator (@imports sub-files)
  palefox-<name>.css      per-area sheets
  userChrome.css          Firefox entry point (only imports palefox.css + user.css)
  user.css                user-owned, not overwritten

docs/dev/                cross-session context — see docs/dev/README.md
docs/                    user-facing site + docs

build.config.ts / build.ts   entry list + bun build wrapper
tests/integration/<area>.ts  Tier 3 Marionette suites
tools/test-driver/           Marionette runner, profile setup
```

`chrome/JS/*.uc.js` are generated; edit `src/` and `bun run build`.

**[docs/dev/](docs/dev/README.md) is the source of truth for cross-session
context.** Plans, architectural rationale, post-mortems, and dissertations
live there — read the relevant file end-to-end before starting related
work. Don't try to recover this from chat history.

## Workflow

```bash
bun run dev         # build + watch (Bun's --watch on the build script)
bun run build       # one-shot production build
bun run typecheck   # tsc --noEmit (do this before committing big edits)
bun test            # pure-function unit tests (Tier 1 — see docs/dev/testing.md)
```

`bun run dev` is the default loop. After edits, the `.uc.js` is rebuilt; reload
Firefox to test. **Type errors do NOT fail the build** (`bun build` doesn't
typecheck) — run `bun run typecheck` separately. Editor tsserver also runs.

### Testing strategy

Palefox's test infrastructure is built in tiers. **Read
[`docs/dev/testing.md`](docs/dev/testing.md) end-to-end before doing
test-related work** — it's the source of truth for the plan, the sprint
checklist, and the guiding principles. Quick orientation:

- **Tier 1** (live): `bun test src/` — pure-function unit tests (persist, helpers)
- **Tier 2** (live): happy-dom-backed mocks of chrome globals — compact state machine
- **Tier 3** (live): real Firefox via Marionette — `bun run test:integration`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tompassarelli/palefox](https://github.com/tompassarelli/palefox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
