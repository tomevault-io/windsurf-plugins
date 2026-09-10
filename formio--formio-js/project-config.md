---
trigger: always_on
description: - **Path:** `packages/formio.js`. **Published as:** `@formio/js` on npm (MIT) — external consumers exist beyond the workspace.
---

# @formio/js

## Identity

- **Path:** `packages/formio.js`. **Published as:** `@formio/js` on npm (MIT) — external consumers exist beyond the workspace.
- **Module system:** Dual CJS + ESM build (`tsconfig.cjs.json` + `tsconfig.mjs.json`). `main` points at `lib/cjs/index.js`. **Language:** JavaScript with TypeScript type-checking (`check-types: tsc --noEmit`).
- **Era:** modern ES-module class era — `export default class X extends Y`, class fields, lodash everywhere, ES `import`. Pockets of CJS in build tooling (e.g. `test/updateRenders.cjs`).
- **Purpose:** The browser SDK / form renderer / form builder for form.io. Defines `Webform`, `Wizard`, `Form`, `FormBuilder`, the `Component` base class, all 40 component types, the `Templates` / `Providers` / `Builders` / `Displays` registries, and the embed/CDN entry points. Consumes `@formio/core` (data processing) and `@formio/bootstrap` (default theme).

## Floor — immutable musts

- **This package is published publicly on npm.** Do not put license-gated logic, secrets, or internal-only fixtures here.
- **Do not clobber `this._submission` or `this._data` upstream of `onSetSubmission` in `Webform`.** They are reference-identity-bearing across [`Webform`](./src/Webform.js), [`Wizard`](./src/Wizard.js), and every `Component`. The correct fix for "setSubmission mutates the caller's submission" is to break aliasing at [`Webform.js:865`](./src/Webform.js) (`submission.data = this.data`), not to reset the internal references. See gotcha [`formio.js/submission-identity-01`](../../docs/gotchas/formio.js.md#submission-identity-01).
- **PRs that touch [`Webform.js`](./src/Webform.js), [`Wizard.js`](./src/Wizard.js), or [`components/_classes/component/Component.js`](./src/components/_classes/component/Component.js) must surface one concern at a time.** These are 1800 / 1200 / 4600-LOC mega-classes; mixing a typo fix, a `return;` adjustment, and a behavior change in one diff hides the load-bearing change at review. See [`formio.js/mega-class-cascade-01`](../../docs/gotchas/formio.js.md#mega-class-cascade-01).
- **A change to `Webform.setSubmission` / `_submission` / `_data` lifecycle requires a Wizard sub-form regression test.** [`Wizard.js:133`](./src/Wizard.js) reads `subForm._submission` directly; [`:1012`](./src/Wizard.js) sets its own. Wizard tests live in [`test/unit/Wizard.unit.js`](./test/unit/Wizard.unit.js) and [`test/wizards/`](./test/wizards/).
- **A change to component rendering (`Component.render`, theme partials, `Display`) must be exercised through the PDF render path too.** See gotcha [`formio.js/pdf-render-path-diverges-01`](../../docs/gotchas/formio.js.md#pdf-render-path-diverges-01).
- **New unit-test files must match `test/unit/*.unit.js`.** `.mocharc.json` globs to that exact pattern and has `bail: true` — subdirectories silently don't run. See [`formio.js/test-discovery-01`](../../docs/gotchas/formio.js.md#test-discovery-01).
- **Do not add new imports from `@formio/core/experimental`.** [`src/templates/Templates.js:2`](./src/templates/Templates.js) already depends on it; widening that surface couples our stability to core's unstable directory. See [`formio.js/templates-experimental-import-01`](../../docs/gotchas/formio.js.md#templates-experimental-import-01).

## Ceiling — emerging patterns

- **Pattern: a new component subclasses `Input` / `Field` / `NestedComponent` (not `Component` directly), exports a default class, and pairs with a `<Name>.form.js` schema file. Example:** [`src/components/textfield/TextField.js`](./src/components/textfield/TextField.js) + [`TextField.form.js`](./src/components/textfield/TextField.form.js) — the smallest reference shape. Register in [`src/components/index.js`](./src/components/index.js).
- **Pattern: registries are static classes consumed via `Formio.<Registry>` at the top of [`src/formio.form.js`](./src/formio.form.js).** Example: [`src/components/Components.js:60`](./src/components/Components.js) (`Components.setComponent(name, comp)`), [`src/providers/Providers.js`](./src/providers/Providers.js) (`addProvider(type, name, provider)`), [`src/templates/Templates.js`](./src/templates/Templates.js). Mirror the static-method shape for any new registry.
- **Pattern: form / submission lifecycle is anchored in [`Webform.js`](./src/Webform.js); Wizard / PDF specialize via subclass.** Example: `Webform` extends `NestedDataComponent`; [`Wizard`](./src/Wizard.js) and [`PDF`](./src/PDF.js) extend `Webform`. Lifecycle methods (`init`, `setValue`, `getValue`, `setSubmission`, `onSetSubmission`) cluster at [`Webform.js:700-910`](./src/Webform.js); read this range whenever the bug touches submission semantics.
- **Pattern: unit tests use a global `jsdom-global` + `Formio.createForm(document.createElement('div'), formJson)` harness. Example:** [`test/unit/Webform.unit.js`](./test/unit/Webform.unit.js) — mid-file (search for `Should handle multiple set submissions`-style tests). Mocha is configured via [`.mocharc.json`](./.mocharc.json) with `bail: true`.

## Blast radius


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [formio/formio.js](https://github.com/formio/formio.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
