---
trigger: always_on
description: Project context for Claude Code and other agents. Keep this file focused on things that are **not** obvious from reading the repo — anything you can grep for in five seconds doesn't belong here.
---

# CLAUDE.md

Project context for Claude Code and other agents. Keep this file focused on things that are **not** obvious from reading the repo — anything you can grep for in five seconds doesn't belong here.

Markdown in this repo is not manually wrapped. Write one paragraph per line and let the editor soft-wrap.

## Layout

- [packages/vite-plugin-cjs-interop/](packages/vite-plugin-cjs-interop/) — the published library (`vite-plugin-cjs-interop` on npm). Built with tsdown. The package name is bare (unscoped) — an intentional exception that predates any "always scope under `@<org>/`" convention you may have seen in similar repos. Don't rename.
- [tests/bench-vite5/](tests/bench-vite5/), [tests/bench-vite6/](tests/bench-vite6/), [tests/bench-vite7/](tests/bench-vite7/), [tests/bench-vite8/](tests/bench-vite8/) — one end-to-end bench package per supported Vite major. Each runs a 4-quadrant matrix: SSR × {build, dev} + client × {build, dev}. SSR builds use `node dist/index.js`; SSR-dev uses `ssr-dev.js` which calls `viteServer.ssrLoadModule`; client cases are driven by `src/ci.test.ts` (vitest + puppeteer) against `vite build` + `vite preview` and `vite dev`. Each bench injects the plugin (`dependenciesMeta.vite-plugin-cjs-interop.injected: true`) so the plugin's `import { Plugin } from "vite"` resolves against the bench's own pinned vite instead of the plugin workspace's devDep. Each bench runs on its own port (`TEST_PORT` env var, 3005–3008) so `pnpm -r run ci` can fan out in parallel without colliding.
- [tests/cjs-test-package/](tests/cjs-test-package/) — fixture CJS package the benches consume (declared as a `file:` dep on its packed `.tgz` so each bench sees it exactly as a published consumer would). Has three variants: `index.cjs` (normal cjs with `__esModule: true`), `wrapped.cjs` (`module.exports.default = {default, named}` with `__esModule` on the inner — the case that defeats Node's `cjs-module-lexer` and forces the plugin to exist), `synthetic.cjs` (named-only, no `__esModule`).

The root [readme.md](readme.md) is a symlink into the package's readme. Edit the symlink target, not the symlink.

## Stack invariants

These are deliberate. Don't change them without a reason.

- **ESM only.** No CJS output, no `"type": "commonjs"`. tsdown is configured for `format: ["esm"]` and `platform: "node"`.
- **Strict TS** with `noUncheckedIndexedAccess` and `noImplicitOverride`.
- **Relative imports use `.ts` extensions**, not `.js`. Lint enforces this; tsconfigs allow it via `allowImportingTsExtensions`. The point is that source runs natively under Node's TS support and Deno, no transpile step required.
- **Tabs, 80 cols.** Markdown and `package.json` use 2-space indent (see [.prettierrc](.prettierrc)). Don't reformat with spaces.
- **Node**: the published source in [packages/vite-plugin-cjs-interop/src/](packages/vite-plugin-cjs-interop/src/) targets the lowest `engines.node` major. The support range covers every active LTS and every Current release. Dev tooling, build configs, and scripts (e.g. `tsdown.config.ts`) can assume the latest minor of the most recent LTS — features that landed in recent LTS minors are fair game there; Current-only features aren't. Off-limits inside the package `src/`.
- **ESLint config** comes from `@cyco130/eslint-config/node`. Lint rules live there, not in-repo.

## Library invariants

vite-plugin-cjs-interop-specific design choices that look like dead code or premature optimization if you don't know why they're there.

- **Transform emits namespace imports + a two-level unwrap chain.** The plugin rewrites `import X, { named } from "cjs"` to `import * as __cjsInterop1__ from "cjs"; const { default: X, named } = __cjsInterop1__?.default?.default?.__esModule ? __cjsInterop1__.default.default : __cjsInterop1__?.default?.__esModule ? __cjsInterop1__.default : __cjsInterop1__;`. The namespace form is mandatory because esbuild's optimizeDeps prebundle (used by vite 7 and earlier on client builds) collapses default-form imports via `_interopRequireDefault` before our wrapper sees them. The two-level chain handles the "wrapped" cjs case (`module.exports.default = {default, named, __esModule}`) plus normal cjs in both Rolldown and esbuild interop styles.
- **`enforce: "pre"` is mandatory.** In Vite's dev pipeline, the built-in `vite:imports-analysis-rewrite` plugin would otherwise rewrite cjs imports to point at `/node_modules/.vite/deps/...` before our filter regex (which matches the original package names) could fire. SSR builds also work under `pre`. Don't change to `post` without re-running the full 4-quadrant matrix.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyco130/vite-plugin-cjs-interop](https://github.com/cyco130/vite-plugin-cjs-interop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
