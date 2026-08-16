---
trigger: always_on
description: **highs-js** compiles the [HiGHS](https://highs.dev/) C++ linear, mixed-integer,
---

**highs-js** compiles the [HiGHS](https://highs.dev/) C++ linear, mixed-integer,
and quadratic optimization solver to WebAssembly. It publishes a dual-format npm
package (`require` and `import`) that runs in Node.js ≥18 and modern browsers.

The package exposes two layers:

1. **Legacy (one-shot) API** — `highs.solve(problem, options)` accepts a model in
   CPLEX LP text format and returns a name-keyed result object. This is the
   compatibility entry point declared as the default export in [types.d.ts](./types.d.ts).
2. **Extended (persistent) API** — `highs.createModel()` / `highs.raw` expose the
   full HiGHS C interface with typed model-building, incremental solves, callbacks,
   basis access, ranging, etc. The runtime implementation lives in the
   [extended runtime source](./src/extended.ts) and the [C++ bridge](./src/highs_js_bridge.cpp).


## Dev Container

The project provides a pre-configured dev container via
[devcontainer.json](./.devcontainer/devcontainer.json) and
[compose.yaml](./compose.yaml). The container installs Emscripten, CMake,
and all npm dependencies, then runs a full build (`npm ci && npm run build`).

Any command that requires Emscripten (such as `npm run build`) should be run
inside the container via `docker compose run --rm tests <command>` if the host
system does not have a working Emscripten installed. The container's entrypoint
sources `/emsdk/emsdk_env.sh` automatically, so no manual setup is needed. If installed, and when changing the container itself, use the `devcontainer` CLI to rebuild the container (`devcontainer [build|up|exec]`).

```sh
# Run a build inside the container
docker compose run --rm tests npm run build

# Run tests inside the container after a build
docker compose run --rm tests npm test

# Check the API manifest inside the container
docker compose run --rm tests npm run check:api
```

---

## Project Layout

| Path | Purpose |
|---|---|
| [HiGHS/](./HiGHS/) | Vendored HiGHS C++ source (git submodule) |
| [src/](./src/) | TypeScript & C++ glue between JS and HiGHS |
| [C++ bridge](./src/highs_js_bridge.cpp) | Custom C++ bridge exposing HiGHS functions to Emscripten |
| [Extended runtime](./src/extended.ts) | TypeScript runtime that decorates the Emscripten module with the extended (persistent) API |
| [Emscripten preamble](./src/pre.js) | Emscripten `--pre-js` preamble |
| [Emscripten postamble](./src/post.js) | Emscripten `--post-js` postamble |
| [Additional type declarations](./src/types/) | Extra type declarations |
| [build/](./build/) | Build output: `highs.js` (CJS), `highs.mjs` (ESM), `highs.wasm`, and generated code |
| [Build script](./build.sh) | Full build script (CMake → Emscripten link → CJS + ESM) |
| [Public type declarations](./types.d.ts) | Public TypeScript declarations for both legacy and extended APIs |
| [api/](./api/) | Audited C API inventory and DTS audit schemas |
| [API manifest generator](./scripts/generate-highs-api.mjs) | Generates the expanded API manifest and export list from the hand-maintained JSON |
| [Package test script](./scripts/test-packed-package.mjs) | Verifies the packed npm package contents |
| [Exported functions](./exported_functions.json) | List of C functions exported to WebAssembly |
| [tsconfig.json](./tsconfig.json) | Type-checks `src/` for JS type errors (no emit) |
| [tsconfig.runtime.json](./tsconfig.runtime.json) | Compiles `src/extended.ts` → `build/generated/extended.js` |
| [Tests](./tests/) | Test suites |
| [Extended tests](./tests/extended/) | Extended (persistent) API tests, each a standalone `node:test` suite |
| [Declaration tests](./test-dts/) | TypeScript declaration tests (compilation-only with `--noEmit`) |
| [Benchmarks](./benchmarks/) | Runtime and size regression benchmarks |
| [compose.yaml](./compose.yaml) | Docker Compose setup for the dev container |

---

## Architecture

The build pipeline works as follows:

1. The [API manifest generator](./scripts/generate-highs-api.mjs) reads
   the hand-maintained [C API manifest](./api/highs-c-api.json), validates it
   against the vendored HiGHS headers, and generates the
   [expanded API descriptor](./api/highs-c-api.generated.json) and the
   [linker export list](./exported_functions.json).
2. **`tsc`** compiles the [extended runtime](./src/extended.ts) via
   [tsconfig.runtime.json](./tsconfig.runtime.json) to
   `build/generated/extended.js`. This keeps the handwritten marshalling code
   type-checkable without modifying the generated loader.
3. **`emcmake cmake`** configures the vendored HiGHS C++ source into CMake.
4. **`emmake make`** compiles HiGHS into object files.
5. **`emcc`** links the object files with the glue code into two loaders:
   - `build/highs.js` — CommonJS (MODULARIZE=1)
   - `build/highs.mjs` — Native ES module (MODULARIZE=1 + EXPORT_ES6=1)

   Both link passes inject the [preamble](./src/pre.js), the
   [marshalling postamble](./src/post.js), and the compiled
   [persistent API layer](./build/generated/extended.js) via `--pre-js`/`--post-js`.
   They share the same object files and flags, so `build/highs.wasm` is identical
   after either pass.

---

## Test Suites

### Legacy tests ([tests/test.js](./tests/test.js))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lovasoa/highs-js](https://github.com/lovasoa/highs-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
