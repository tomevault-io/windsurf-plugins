---
trigger: always_on
description: Firefox MV2 WebExtension + Experiment API for Zen Browser. Command palette for tab management.
---

# ErgoZen

Firefox MV2 WebExtension + Experiment API for Zen Browser. Command palette for tab management.

## Source Root

`src/` is the extension source. It builds into the generated `dist/` extension root with `npm run build`; Zen should load `dist/manifest.json`.

Do not edit generated `dist/` files.

## Architecture

Three execution contexts — they cannot share code or globals:

1. **`src/experiment/api.js`** — Chrome-privileged parent process. Full access to `gBrowser`, `gZenWorkspaces`, `gZenMods`, chrome DOM. Exposes `browser.zenWorkspaces.*` API. Also creates the palette overlay.
2. **`src/background.js`** — Extension background script. Routes messages between popup and experiment API. Owns auto-close/auto-move logic.
3. **`src/popup/`** — Content process inside a XUL `<browser>` element. No chrome access. Communicates via `browser.runtime.sendMessage` to background.js.

## Architecture Guardrails

The current design intentionally separates capture, sequence, model, and render
ownership. Keep new features inside these boundaries:

- **ChordSession owns chord progression.** `src/experiment/chord-session.js`
  is the only chord-tree traverser and owns replay traces, bridge state, leader
  timing, reveal deferral/blocking, and state invariants. Do not add a second
  chord matcher in popup, background, or content scripts.
- **Physical-key ingress owns exactly-once key consumption.**
  `src/experiment/chord-key-ingress.js` is the boundary every chrome-side
  keystroke path crosses before it reaches `ChordSession.acceptKey`: chrome
  shim, content shim, root fallback, and hidden-chain fallback all submit
  normalized key data through `ingress.submit(keyData, source)`. No path should
  call `acceptKey` or `forwardKeyToPopup` directly to handle a physical key.
- **Shims only capture keys.** `src/shared/chord-shim.js` and the chrome/content
  listeners synchronously suppress capturable keydowns and forward normalized
  keys to `ChordSession`. They should stay stateless apart from armed/disarmed
  and their fail-safe timeout.
- **Overlay lifecycle state lives in `overlay-controller`.** Popup instance
  gating, pending reveal closures, morph generations, current view params, nav
  stack, and resize diagnostics belong to `src/experiment/overlay-controller.js`.
  `api.js` should provide the chrome/XUL implementation details; do not scatter
  popup-instance or pending-reveal state back into top-level globals.
- **Popup readiness gating is a named boundary.**
  `src/experiment/popup-readiness-guard.js` owns
  `acceptsPopupViewStateMessage(deps, inst, readyGen)`, the authority for
  whether inbound popup-to-chrome state writes are accepted. Visible mouse
  navigation may omit `readyGen` only when there is no active bridge and no
  pending reveal.
- **Chrome view transitions go through the coordinator.**
  `createChromeViewTransitionCoordinator` in `src/experiment/api.js` owns every
  write to `overlayController.setCurrentView` / `resetViewState`, every
  `chordSession.retargetActiveBridgeView`, every `armRevealTimer`, and every
  `WarmRearm` send. Enforcement lives in
  `src/experiment/chrome-view-transition-boundary.test.ts`, which strips the
  coordinator body and greps the remaining source for forbidden call shapes.
- **Chrome owns authoritative view models where migrated.** Recents, tab lists,
  domains, workspaces, folders, containers, profiles, duplicates, actions, and
  duplicate-prompt tab rows are driven by chrome-side DTO/model APIs. The popup
  renders DTOs and reports row intents with stable row IDs and sequence/list
  versions. Do not rebuild a parallel recents/tabs/domains/workspaces model in
  Svelte.
- **Popup owns visible UI interaction only.** Structural UI keys such as arrows,
  Enter, Tab, Space, Escape, Backspace, sort/filter toggles, preview, and close
  hints live under `src/popup/interaction/` and `src/popup/runtime/`. Prefer
  pure helpers with tests for decisions, with `NativePalette.svelte` acting as
  wiring for stores/effects/lifecycle.
- **Background remains the runtime router.** `src/background.js` routes
  WebExtension messages and executes runtime actions. It should not maintain
  chord replay state; call `recordRuntimeActionForReplay` and let
  `ChordSession` decide whether to promote an in-flight chord trace or store a
  raw runtime action.
- **Never duplicate model ownership for convenience.** If chrome needs to
  resolve a key, move that view's model to chrome and send compact DTOs to the
  popup. If the popup still owns a special-case UI view, keep chrome's knowledge
  transitional and explicit. Two independently computed row orders are a bug
  class, not an optimization.

### Boundary Enforcement Pattern

When a new owner boundary is introduced, ship the enforcement test in the same
commit. The two cheapest patterns are source-grep tests that forbid bypassing a
coordinator and small pure modules, like `popup-readiness-guard.js`, that can be
loaded directly in unit tests with injected dependencies.

## Critical Gotchas


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cfilipov/ergozen](https://github.com/cfilipov/ergozen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
