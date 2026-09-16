---
trigger: always_on
description: JSON Semantic Diff is a local-first JSON comparison utility. Its defining behavior is to explain meaningful structural/value changes while matching reordered object arrays by inferred row identity when evidence is strong enough.
---

# AGENTS.md — JSON Semantic Diff

## Product intent

JSON Semantic Diff is a local-first JSON comparison utility. Its defining behavior is to explain meaningful structural/value changes while matching reordered object arrays by inferred row identity when evidence is strong enough.

## Non-negotiable product principles

- JSON content must remain in the browser. Do not add network calls that upload, persist, log, or analyze user JSON.
- Identity inference must remain deterministic and explainable. Do not add ML/AI models to the diff path.
- Never silently guess an identity when confidence is low or candidate scores are ambiguous. Fall back to position and expose the analysis.
- Keep the primary workflow one-page: paste/drop two JSON documents, compare, inspect results, all on a single route (`{ path: '', component: AppComponent }`). A second, static informational/marketing page (`/how-it-works`, no comparison workflow of its own) is permitted alongside it — this is the only sanctioned exception. Do not add a third route, and do not add any interactive comparison functionality outside the `''` route, without deliberately revisiting this principle first.
- Avoid IDE-like chrome. No permanent console or settings sidebar for V1.
- Advanced detail should be progressive: inline match badges -> analysis drawer.
- Preserve responsive desktop-first behavior; JSON comparison is optimized for laptop/desktop widths.

## Repo layout

This is an npm-workspaces monorepo:

- `packages/core` — the framework-free diff engine. Published to npm as `json-semantic-diff`. Must not import Angular, RxJS, or DOM APIs. Testable and buildable standalone (`npm run build`/`npm test` from within `packages/core`).
- `packages/ui` — the Angular app. Consumes `packages/core` via the npm workspaces symlink (`"json-semantic-diff": "*"`) and, for local dev/CI builds, via a `paths` alias in `packages/ui/tsconfig.json` that resolves straight from `packages/core/src` — no build-ordering step is required to develop the app.

This alias causes `ng build`/`ng test` to print `File '...' not found in TypeScript compilation.` warnings for every `packages/core/src` file pulled in transitively (and for a handful of `packages/ui/src` files reached only via a `.spec.ts` import). This is expected, upstream Angular CLI behavior for `paths`-aliasing a sibling workspace package's raw source (confirmed via `angular/angular-cli#27176` — Angular's esbuild builder does not treat it the way plain `tsc` `include` would, and widening `tsconfig.app.json`/`tsconfig.spec.json`'s `include` does not suppress it). It is **not** a bug and does not need fixing: `packages/core` is still fully type-checked independently by its own `tsc`/Vitest run, which the root `npm run build`/`npm test` scripts always run first. Don't "fix" this by touching `include`/`exclude` in `packages/ui`'s tsconfigs — it won't work, and isn't the supported path anyway (the supported fix would be dropping the `paths` alias and consuming `packages/core`'s built `dist/` output through plain npm workspace resolution instead, which reintroduces the build-ordering step this alias exists to avoid).

## Runtime and framework

- Angular: 22.x
- Node: pinned via `.nvmrc` (currently `v24.17.0`) — this is the single source of truth for the required Node version (all CI workflows read it via `node-version-file`). There is deliberately no `engines` field in the root `package.json` (removed; it duplicated `.nvmrc` and could drift out of sync with zero functional benefit — see git history).
- TypeScript: 6.0.x
- UI primitives: Angular CDK where interaction primitives are required.
- Styling: Tailwind CSS 4 plus application CSS variables/components. Tailwind utilities are used sparingly, mainly on the `home`/`how-it-works` marketing-style surfaces; most components use hand-rolled component CSS by design. Don't Tailwind-ify existing component styles without a clear reason. Do not introduce a second full visual component system without a clear need.
- State: Angular signals. Prefer local/component state over global stores until cross-feature state actually warrants one.

## Prerendering

`packages/ui` builds with build-time-only static prerendering (`angular.json`'s `outputMode: "static"` + `server: "src/main.server.ts"` + `@angular/ssr`'s `RenderMode.Prerender` in `app.routes.server.ts`) — no live Node server ships to production, this is purely so the deployed site's initial HTML contains real rendered content instead of an empty `<app-root>` (for crawlability/SEO). `ng build` runs this prerender pass in a Node environment with **no browser globals** (`window`/`document`/`localStorage`/`navigator` do not exist).

**Any new code that reads a browser global must guard it**, or a future `ng build`/deploy can crash or silently produce broken output:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cchandurkar/json-semantic-diff](https://github.com/cchandurkar/json-semantic-diff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
