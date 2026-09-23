---
trigger: always_on
description: A local-first events board for AI coding agents. One board for every agent,
---

# signalbox

A local-first events board for AI coding agents. One board for every agent,
terminal, and job you run.

## Specs are the source of truth - keep them current

**Whenever you change behaviour, update the spec in the same change.** The specs
in `components/specs/` describe the contract; they must never lag the code.

- `components/specs/cli.md` - every CLI command, flag, and its output. Update it
  when you add/rename/remove a command or flag, or change what a command prints.
- `components/specs/events.md` - the wire schema (event types, fields, reducer
  rules). Update it when the event shape or reducer behaviour changes.
- `components/specs/adapters.md` - how each agent adapter fires events.
- **`components/specs/*.html` are the living spec for the app's UI surfaces** -
  the HTML mock IS the source of truth for that surface, not just an
  illustration:
  - `components/specs/settings.html` - the Settings window (every control, its
    label, caption, and the settings-storage table). Change a setting -> update
    this.
  - `components/specs/hub-jumplist.html` - the jumplist (rows, keys, footer,
    marks).
  - `components/specs/menubar.html` - the menu bar icon + dropdown.
  When you add/change/remove a control or behaviour on one of these surfaces,
  update its HTML mock in the same change.

If a change touches behaviour and you did not touch a spec, that is a bug in the
change. Treat "code and spec disagree" as a failing state.

## Layout

- `components/cli/` - the TypeScript CLI + hub, compiled to a single binary with
  Bun (`bun build --compile`). The hub is `signalbox hub` (same binary).
- `components/app/` - the Swift macOS menu bar app (jumplist, status icon,
  settings). The app OWNS the hub: it spawns `signalbox hub` as a child,
  keeps it alive, and stops it on quit (Hub.swift) - there is no LaunchAgent.
  The bundle embeds the CLI at Contents/Resources/signalbox. Built via
  `components/app/Makefile` (it works around a CommandLineTools SPM manifest
  bug - use `make -C components/app app`, not bare `swift build`; the `build`
  target only compiles Swift and leaves a stale CLI in the bundle).
- `components/cli/adapters/` - per-agent hooks/plugins (claude, opencode, pi) and
  tmux.
- `components/scripts/` - dev helpers (e.g. `demo.sh` seeds a board via `fire`).
- `packaging/` - the Homebrew formula template.
- `docs/`, `components/specs/` - docs site and specs.

## Build & test

```bash
make build                     # compile the CLI to components/cli/bin/signalbox
make -C components/app app     # build the menu bar app bundle (embeds the CLI;
                               # plain `build` compiles Swift only - stale bundle)
make -C components/ios device  # cable deploy: stamped build -> USB iPhone
cd components/cli && bun test  # CLI + reducer tests
cd components/cli && bunx tsc --noEmit   # typecheck
```

Larger PRs - anything that changes behaviour across the hub, forwarder,
adapters, or either app - should include an integration evidence run: the
`/integration-test` skill (`.claude/skills/integration-test/`) drives the
build, hub modes, forwarder + spool, hooks, the macOS app, and the iOS app in
the Simulator end to end, and writes a self-contained HTML report with
screenshots to `scratch/integration/`. Note the pass/warn/fail counts in the
PR description.

Cable deploys must use the `device` recipe, never bare `xcodebuild` - it
stamps the build so Settings proves what the phone runs (why: see the
`components/ios/Makefile` header).

`~/.local/bin/signalbox` is symlinked to `components/cli/bin/signalbox`, so
`make build` deploys the CLI. The app supervises the hub: `make install` kills
a running hub and the app respawns it with the new build within seconds;
relaunch the app itself to pick up an app rebuild.

### Regenerating the hero gif

`docs/images/hero-anim.gif` (README + `docs/assets/hero-images/hero.html` on
the landing page) is a rendered capture of `hero.html`'s `.split` element, not
hand-edited. The README embeds it with `<img width="900">`, but the gif's own
pixel dimensions must NOT be 900-wide - see the resolution note below. After
changing `hero.html`, regenerate it:

```bash
# serve the file (file:// is blocked by headless browsers)
cd docs/assets/hero-images && python3 -m http.server 8791 &

# one-off Playwright + gifsicle install, in a scratch dir (gitignored)
mkdir -p scratch/hero-gif && cd scratch/hero-gif
npm init -y && npm install playwright && npx playwright install chromium
brew install gifsicle   # if not already on the machine
```

Capture script (`scratch/hero-gif/capture.js`) - steps through the CSS
animation with `Animation.currentTime` rather than waiting in real time, so
every frame is exact regardless of machine speed. `deviceScaleFactor: 2`
renders at 2x pixel density (retina-equivalent):

```js
const { chromium } = require('playwright');
const path = require('path');
const fs = require('fs');

const OUT_DIR = path.join(__dirname, 'frames');
const URL = 'http://localhost:8791/hero.html';
const DURATION_MS = 9000; // must match the CSS animation's total loop length
const FPS = 12;
const FRAME_COUNT = DURATION_MS / 1000 * FPS;

(async () => {
  fs.mkdirSync(OUT_DIR, { recursive: true });
  const browser = await chromium.launch();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dwmkerr/signalbox](https://github.com/dwmkerr/signalbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
