---
trigger: always_on
description: PCF Workbench is a local development harness that replaces `pcf-scripts start` for Power Apps Component Framework (PCF) controls. It loads a user's compiled `out/controls/<Name>/bundle.js`, runs it against shimmed `ComponentFramework.Context` APIs, and adds gallery, device emulation, network conditioning, WebAPI mocking, scenarios, and lifecycle/leak monitoring.
---

# PCF Workbench — Copilot Instructions

PCF Workbench is a local development harness that replaces `pcf-scripts start` for Power Apps Component Framework (PCF) controls. It loads a user's compiled `out/controls/<Name>/bundle.js`, runs it against shimmed `ComponentFramework.Context` APIs, and adds gallery, device emulation, network conditioning, WebAPI mocking, scenarios, and lifecycle/leak monitoring.

## Working directory

All source, configs, and scripts live under `harness/`. Always `cd harness` first. The repo root only contains README and LICENSE.

## Commands (run from `harness/`)

- `npm install` — install deps (no separate build step needed for dev; Vite transpiles TS on the fly)
- `npm run dev` — start Vite on port 8181
- `npm run typecheck` — `tsc --noEmit`; this is the canonical pre-PR check (there is no test suite or linter configured)
- `npm run build` — `tsc -b && vite build` (production bundle of the harness itself)
- `npm run harness` — runs `bin/pcf-harness.ts` via tsx (CLI entry exposed as `pcf-harness` bin)

Launching against a target control workspace is driven by env vars, not CLI args:

- Gallery mode: `PCF_WORKSPACE_ROOT=<dir-with-many-controls> npx vite --port 8181`
- Single-control mode: `PCF_CONTROL_PATH=<path-to-control-dir> npx vite --port 8181`
- PowerShell uses `$env:PCF_WORKSPACE_ROOT = "..."` etc.

The harness loads the **compiled** `bundle.js` from `out/controls/<Name>/`, never the user's TS source. Users must run `npm run build` in their PCF project before the control is loadable.

## Architecture (the parts that need cross-file reading)

The harness has two halves that meet at the Vite plugin:

1. **Server side — `src/vite-plugin/pcf-plugin.ts`**
   Single Vite plugin owns: workspace/control discovery, `ControlManifest.Input.xml` parsing (via `parser/manifest-parser.ts` + `fast-xml-parser`), serving `bundle.js` and resources from the user's `out/` dir, the gallery JSON API, and a watcher on `out/` that triggers HMR when `bundle.js` changes. There is no separate Express/Node server.

2. **Client side — React + Zustand**
   - `App.tsx` routes between `ui/gallery/Gallery.tsx` (workspace mode) and `ui/HarnessShell.tsx` (single-control mode).
   - `loader/control-host.ts` is the PCF lifecycle manager — it calls `init`, `updateView`, `getOutputs`, `destroy` on the user control and feeds it the shimmed context. Also wires the form state: after `preloadBundleResources` it calls `seedFormState` → `setFormContextLogger` → `buildFormContext` → `bindXrmPageToFormContext`, then `fireOnLoad(buildExecutionContext('form.load', null))` after the first `updateView`.
   - `loader/bundle-loader.ts` injects the compiled bundle as a script and reads the registered control class off the global namespace.
   - `loader/platform-libs.ts` exposes React/ReactDOM as globals for virtual controls and installs Fluent UI v8/v9 Proxy stubs (functional component impls of Stack/TextField/Dropdown/etc.). Treat these stubs as the API contract for virtual controls — extend them when a control needs a missing Fluent component.
   - `loader/resource-tracker.ts` monkey-patches `addEventListener`, `setInterval`, `setTimeout`, and the three Observer constructors before `init` runs and diffs after `destroy` to report leaks.

3. **Context shims — `src/shim/`**
   `context-factory.ts` composes a full `ComponentFramework.Context` by pulling one shim per concern: `web-api.ts`, `resources.ts`, `client.ts`, `device.ts`, `mode.ts`, `navigation.ts`, `formatting.ts`, `user-settings.ts`, `utils.ts`, `fluent-design.ts`. **One file per `context.*` namespace** is the convention — add new context surface area as a new shim file rather than expanding `context-factory.ts`.
   - `web-api.ts` mirrors the real Dynamics 365 routing model: `context.webAPI` auto-routes online/offline based on network state, while `webAPI.online` and `webAPI.offline` always target a specific store. OData support covers `$filter` (eq/ne/gt/ge/lt/le, `contains`/`startswith`/`endswith`, `and`/`or`, null), `$select`, `$orderby`, `$top`, `maxPageSize`.
   - `resources.ts` preloads images/fonts/RESX synchronously at startup so `getResource()` and `getString()` return cached base64/strings the same tick — matching real Dynamics timing. Don't make these async.
   - `xrm-global.ts` installs the global `Xrm.*` namespaces (`WebApi`, `Navigation`, `Utility`, `Encoding`, `Device`, `App`, `Panel`) with best-practice warning wrappers. **One block per Xrm namespace** — add new ones following the same `if (!w.Xrm.X) { w.Xrm.X = {…} }` pattern.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jaduplesms/PCF-Workbench](https://github.com/jaduplesms/PCF-Workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
