---
trigger: always_on
description: handles, bounded and freed with the runner.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

`@hegeldev/hegel` — property-based testing for TypeScript. The client drives
**libhegel**, the Rust engine (`hegel-rust/hegel-c`, version pinned in
`src/libhegel-version.ts`), directly through its C ABI. On Node, Bun and Deno
that is the native shared library via the `koffi` FFI library; in browsers it
is the same engine compiled to `wasm32-unknown-unknown`, driven through the raw
Wasm exports. There is no subprocess and no wire protocol: the engine runs
inside libhegel, and the client calls its functions synchronously. Both
runtimes share one runner, one schema interpreter and one public API.

## Commands

```bash
npm install
just test    # fetch libhegel into native/, vitest with coverage (fails if < 100%)
just lint    # prettier --check + eslint + tsc --noEmit + typecheck:portable
just format  # prettier --write
just docs    # typedoc (treatWarningsAsErrors) + open
just check   # lint + docs + test — everything CI runs on the main matrix

npm run test:packaging  # node --test scripts/packaging.test.mjs (export map, sidecars, update-libhegel)
npm run test:browser    # pack the package, bundle it four ways, run in Chromium (see below)
```

- Single file: `npx vitest run tests/integers.test.ts`; by name:
  `npx vitest run -t "test name"`. Omit `--coverage` — the 100% threshold only
  holds for the full suite.
- Tests load the real engine, native and Wasm. Run `just fetch-libhegel` once
  to download both into `native/<version>/`, or set `HEGEL_LIBHEGEL_PATH`
  (resolved by `tests/libPath.ts`) and `HEGEL_WASM_PATH` (resolved by
  `tests/wasmFixture.ts`).
- `just build-libhegel` builds libhegel from a sibling `../hegel-rust` checkout
  for work against an unreleased engine; export the printed path as
  `HEGEL_LIBHEGEL_PATH`. For the Wasm side, build there with
  `cargo build -p hegeltest-c --release --target wasm32-unknown-unknown` and
  export the module's path as `HEGEL_WASM_PATH`.
- `npm run test:browser` needs the fixture in `tests/browser/` installed first:
  `(cd tests/browser && npm ci && npx playwright install chromium)`. The
  bundlers and Playwright are devDependencies of that fixture, not of the
  library.

## Engine Distribution

Each platform's native libhegel ships as its own npm package
(`@hegeldev/hegel-<os>-<arch>`), exact-pinned in `optionalDependencies`; the
`os`/`cpu` fields in each manifest make package managers install only the
host's package. `scripts/make-platform-packages.mjs` assembles them into
`platform-packages/`, and the release pipeline publishes all five before the
main package.

Runtime resolution (`src/locate.ts`, synchronous by necessity):
`HEGEL_LIBHEGEL_PATH` → the platform package's `./binary` export. There is no
`native/` step in production code: the repo's own test runs point
`HEGEL_LIBHEGEL_PATH` at the pinned engine (`just check-test` wires this up
from `fetch-libhegel`'s output).

The Wasm module (`libhegel-wasm32-unknown-unknown.wasm`) ships inside the main
package at `dist/browser/`, next to the browser entry: `npm run build` is
`tsc && node scripts/fetch-libhegel.mjs --wasm dist/browser`, and `prepack`
runs it. `src/browser/index.ts` loads it with a static
`new URL("./libhegel-wasm32-unknown-unknown.wasm", import.meta.url)` so Vite
and webpack emit it as an asset; esbuild and Rollup users copy the
`./libhegel.wasm` export beside their bundle (`BROWSER.md` documents this).
The root export map is ordered `types` → `node` → `browser` → `default`, so
Node wins whenever it is enabled and browser-oriented tools cannot fall back to
the koffi entry; `scripts/packaging.test.mjs` pins that ordering.

`src/libhegel-version.ts` is the single pin for both artifacts.
`scripts/fetch-libhegel.mjs` downloads the pinned release's host library and
Wasm module into `native/<version>/`, verifying each against the release's
`.sha256` sidecar (the versioned path is the cache invalidation — a pin bump
misses and re-downloads). Regenerate the pin with `just update-libhegel`; it
refuses a release that does not publish every platform asset, the Wasm module
and their sidecars. The pin file is generated code — never edit by hand. After
a bump, follow the `align-libhegel` skill: the koffi bindings _and_ the raw
Wasm signature table in `src/browser/abi.ts` must be audited against `hegel.h`.

## Architecture

Layers, each building on the previous. Everything above the engine adapters is
shared by Node and the browser; `tsconfig.portable.json` type-checks exactly
that shared set plus `src/browser/` against `ES2022 + DOM` with no Node types,
so a Node import creeping into a shared module fails `just lint`.

1. **Engine interface** (`src/engine.ts`) — `Engine` is the runtime-independent
   view of the C ABI: one method per ABI operation, opaque branded handle types
   (`ContextHandle`, `TestCaseHandle`, …) so pointers from different adapters
   cannot be mixed, the `Status`/`RunStatus`/`NativeVerbosity` enums copied
   from `hegel.h`, and `EngineError` for adapter faults that must never reach
   the shrinker.
2. **Native adapter** (`src/locate.ts`, `src/libhegel.ts`, `src/session.ts`) —
   `locate.ts` resolves the shared library as described above; `Libhegel`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hegeldev/hegel-typescript](https://github.com/hegeldev/hegel-typescript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
