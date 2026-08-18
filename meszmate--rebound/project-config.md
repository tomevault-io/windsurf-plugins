---
trigger: always_on
description: Rebound is a free **After Effects panel** for easing, springs, and
---

# Rebound, contributor & agent guide

Rebound is a free **After Effects panel** for easing, springs, and
motion-design workflow tools. It is a buildless Adobe **CEP** extension: an
HTML/CSS/JS panel that talks to an **ExtendScript** host.

This file is the source of truth for how the project is built. `CLAUDE.md` is a
symlink to it.

---

## Principles

1. **Original, clean-room code.** Everything is implemented from first
   principles against *public* specifications, CSS cubic-bezier timing, the
   Penner easing equations, spring physics (a damped harmonic oscillator), and
   Adobe's documented ExtendScript / CEP APIs. We do **not** copy any other
   product's source, scripts, assets, preset names, or interface. The only
   vendored third-party file is Adobe's `client/js/lib/CSInterface.js`.
2. **Describe features by what they do.** In code, comments, and docs, name a
   feature by its behavior (e.g. "physical spring easing", "anchor-point
   repositioning"), not by any other tool. Keep competitor brand names out of
   the codebase.
3. **Buildless.** No bundler. Plain files loaded in dependency order. The same
   pure-logic modules run in the panel *and* under Node tests.
4. **Non-destructive by default, bakeable on demand.** Prefer native temporal
   ease and live expressions; always offer a clean bake. Always tell the user
   which mode was used.
5. **A Home widget is ONE simple control, not the whole tool.** When a tool is
   embedded as a Home widget it shows a single primary control, the interactive
   heart of the tool, filling the box edge to edge, exactly like the Ease widget
   (just its curve) and the Anchor widget (just its 9-point stage). A widget is
   never the tool's full control panel shrunk into a box. Secondary options are
   not crammed in next to it; they live in the full tool, one click away via the
   widget's open control, so they are easy to reach without taking space or
   bothering the main control. See "Home & widget UX" below.

---

## Repository layout

```
CSXS/manifest.xml        CEP manifest, declares the panel extension
.debug                   CEP remote-debug port (panel 8718)

client/                  the panel (HTML/CSS/JS), runs in CEP's Chromium
  index.html             main panel; lists every script in load order
  settings.html          settings/preferences extension
  css/                   design system (base, components, layout, curve-editor)
  js/
    lib/CSInterface.js   vendored Adobe bridge library
    core/                bridge, theme, store, registry, dom, events, units, log
    easing/              pure math: bezier, penner, spring, sampler (UMD, tested)
    presets/             built-in preset library
    ui/                  controls + the curve editor widget
    features/            one file per tool (registers itself)
    main.js              shell bootstrap (loaded last)

host/                    ExtendScript host (ES3), runs in After Effects
  index.jsx              entry; evaluates lib + command modules in order
  lib/                   json (polyfill), core (RPC dispatch), util (helpers)
  commands/              one file per command group; registers handlers

docs/                    FEATURES, INSTALL, DEVELOPMENT, ARCHITECTURE
test/                    Vitest unit tests for the math + units core
tools/                   Node dev tooling (icons, cert, pack, install, debug)
```

---

## Architecture in one screen

- **Bridge (`client/js/core/bridge.js`)**, the only module that touches
  CSInterface. `Rebound.bridge.invoke(method, args)` returns a Promise of a host
  command's result. evalScript is async on Windows, sync on macOS, and always
  returns a string, so every call goes through a JSON envelope.
- **Host RPC (`host/lib/core.jsx`)**, `$.__rebound.dispatch(method, argsJson)`
  returns `{"ok":true,"data":…}` or `{"ok":false,"error":…}`. Commands
  registered with an undo label run inside one `beginUndoGroup`/`endUndoGroup`.
- **Tool registry (`client/js/core/registry.js`)**, each feature calls
  `Rebound.tools.register({ id, title, group, mount })`. The shell builds
  navigation from the registry and mounts a tool on demand.
- **Easing engine (`client/js/easing/*`)**, pure, host-agnostic, UMD-wrapped
  so it runs in the panel and imports into Vitest. New physics belongs here so
  it stays unit-testable; the host only receives resolved values/eases to write.
- **Reactive store + theme**, a tiny framework-free store drives all views;
  theme reads the host skin and sets CSS custom properties.

### Module pattern (panel)

Every panel script is an IIFE attaching to the global namespace:

```js
;(function (R) {
  'use strict';
  // ...
  R.something = ...;
})(window.Rebound = window.Rebound || {});
```

Pure-logic modules (easing, units) additionally use a UMD header so Vitest can
import them. They register their dependencies from `Rebound.*` first and only
fall back to `require()`.

### Host command pattern

```jsx
(function () {
  var R = $.__rebound;
  R.register('group.action', function (args) {
    // ... do work, return a plain JSON-serialisable value ...
    return { ok: true };
  }, 'Rebound: Action');   // <- undo-group label (omit for read-only commands)
})();
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meszmate/rebound](https://github.com/meszmate/rebound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
