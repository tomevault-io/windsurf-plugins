---
trigger: always_on
description: Three_Slicer — a browser/WASM slicer reverse-engineered from OrcaSlicer. The root holds three folders:
---

# AGENTS.md

Three_Slicer — a browser/WASM slicer reverse-engineered from OrcaSlicer. The root holds three folders:

- **`slicers/`** — the upstream reference checkouts, untracked: OrcaSlicer at `slicers/slicer` (the extraction/porting source — its own guide is `slicers/slicer/AGENTS.md`) and PrusaSlicer at `slicers/PrusaSlicer` (comparison only).
- **`packages/`** — the published npm package `three-slicer` (AGPL) plus the kernel sources. Zero build or runtime dependency on `slicers/`.
- **`viewer-package/`** — the published npm package `three-slicer-viewer` (MIT): the viewer without the slicer — model loading, G-code parsing, GPU toolpath rendering, the catalog-free settings transforms, the toggle evaluator (unbound), `<SettingsPanel/>` and `<Viewport/>` themselves. `three-slicer/viewer` is a thin wrapper that plugs the kernel and the vendor catalog into it. The two publish as a locked pair (same version, exact pin — `packages/RELICENSE.md`).
- **`web/`** — the demo app shell. It consumes the package as a workspace (no relative-path imports). Details: `web/README.md` (the stage log is split out as `web/HISTORY.md`), `web/GUIDE.md`, `web/SPECS.md`.

The root `package.json` is the npm workspaces root (`viewer-package`, `packages`, `web/viewer`) — a single `npm i` at the root installs everything. `viewer-package` comes first because `packages` depends on it.

## Core rules

- **Never modify `slicers/`.** All development happens in `packages/` and `web/`.
- **No ternary operator (`cond ? a : b`), in any language or test.** Not nested, not single, not in JSX. A ternary
  packs a branch into an expression, where the condition, the two values and their order all have to be read at
  once — and a single one invites the next one inside it. Write an early `return`, an `if`/`else` assignment, a
  lookup table with `??` when the choice maps names to values (`{ key: value }[name] ?? fallback`), and `cond &&
  <Element/>` for conditional JSX. `?.` and `??` are not ternaries and stay. Existing code is converted as it is
  touched.
- **A value that has a named owner is read from it, never typed out again.** A list, label, limit or conversion
  written a second time is right only until the owner changes, and nothing fails when the copy drifts. The format
  list is the case that happened: `SUPPORTED_EXT` (`scene/model_loaders.js`) grows through `registerLoader()`, while
  the drop overlay and the load-rejection message spelled out `STL/OBJ/3MF/AMF/PLY`, so neither named STEP, which
  the demo registers, and the overlay named neither SL1 nor G-code. Derive the text from the owner (`EXT_LABEL` in
  `Viewport.jsx`, `SUPPORTED_EXT` in `model_load.js`); where a value has no owner yet, make one constant and read it
  everywhere. `test_layers.mjs` fails on the format list spelled out in code.
  A plain `''` or `0` has no owner and never changes, so it is not a constant. An empty value that carries a
  meaning is named instead: "clear the message" is `clearError()` / `clearSliceNotice()` / `clearTriWarn()`
  (`Viewport.jsx` wiring), not `setError('')` at each call site, and "no value" is `null`. `test_layers.mjs` fails on
  a `set…('')` outside those definitions.
- `packages/` and `web/` must run, build and publish without `slicers/` (demonstrated in stage 34). Do not make changes that break this independence.
- Changes to the kernel (`packages/wasm-core/`) must pass the golden byte-identical check (`golden.mjs`) and the `test.mjs` invariant suite.
- Multi-material widened what "byte-identical" has to cover. Three conditions, each with its own `test.mjs` invariant, must keep producing the output the kernel produced before the feature existed: **no painted facets**, **no per-extruder arrays** (`extruder_nozzle_temp`, `extruder_flow_ratio`, `extruder_retract_*`, `extruder_z_hop`), **`support_filament` 0**. All three hold by omission rather than by a default: `deriveKernelParams` leaves those keys out of the params object entirely (93 keys from an empty settings map today), and `Params::forTool` / `support_tool_of` fall back to the scalar and to "emit no `T` command at all".
- **Omission is now the general rule, not a multi-material special case.** The kernel reads 159 parameters and
  `deriveKernelParams` can produce 131 of them; the `PASSTHROUGH_*` lists in `settings.js` are the ones whose schema
  key carries the same name, and every one is emitted **only when the settings map actually holds the key**. Filling
  in a schema default would be wrong rather than merely noisy: the two defaults disagree for several keys
  (`independent_support_layer_height` is true in the schema and false in the kernel, `printable_height` 100 vs 250),
  so a default-filled param would silently reslice every existing caller's model. The empty-map key count is the
  invariant that catches a violation. Two traps live in that list: `support_line_width` is `coFloatOrPercent` but is
  read by the kernel's plain number reader, not the percent-aware `jwidth_raw` the per-feature widths get, so a
  `"120%"` would reach `strtod` as a quoted string and resolve to 0 == auto — it is resolved against the nozzle on

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kimgh06/Three_Slicer](https://github.com/kimgh06/Three_Slicer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
