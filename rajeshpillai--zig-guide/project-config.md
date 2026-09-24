---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A Zig tutorial site (Astro) where **every code snippet is compiled and executed by CI against current Zig master**, and where the *same* `.wasm` artifact CI ran is what the reader executes in their browser. There is deliberately no second code path — the docs *are* the code. Silent staleness is the one bug class this project cannot tolerate.

Requires **Zig master** (tracks master, not a stable release) and **Node 24+**. Node 22 segfaults inside `node:wasi` on about 1% of runs of the `06-cookbook.serialization-size` snippet, which made the nightly fail on an unchanged tree; CI pins 24 for that reason and the comment in `ci.yml` records the measurement.

## Commands

```bash
./dev-start.sh              # the whole dev loop: build snippets, watch, serve on :4321
./dev-start.sh --verify     # also run the full CI gate before serving and on each change
```

By hand — **order matters**, `zig build` must precede any web build:

```bash
zig build verify            # CI gate: compile + run + diff stdout for every snippet
zig build                   # emit wasm + snippets.json into web/public/wasm/
zig build -Doptimize=Debug  # when you need stack traces in a snippet
cd web && npm install && npm run dev
cd web && npm run build     # astro build
cd web && npm run e2e       # headless-Chromium gate: run every playground, check every link
cd web && npm run check     # astro check
```

There is no per-test runner: `zig build verify` walks all of `snippets/` in one step. To exercise a single snippet, run its wasm directly:

```bash
node tools/run-wasi.mjs web/public/wasm/02-language.optionals.wasm
```

`zig run snippets/<chapter>/<file>.zig` compiles and runs one snippet without the build graph, which is the quick loop while writing one. It does **not** run the deprecated-name gate, so a snippet using `std.mem.indexOfScalar` passes it and fails `zig build verify`. That gate runs standalone and touches no build cache, which also makes it the one to use when something else is holding the cache:

```bash
node tools/check-deprecations.mjs "$(which zig)" "$(zig version)" ./snippets/<chapter>/<file>.zig ./web/src/content/docs/<section>/<file>.mdx
```

The game is a separate project with its own build (see [The game](#the-game-exampleslane-dodger)):

```bash
cd examples/lane-dodger
./fetch-raylib.sh                      # once: vendor raylib's C at a pinned commit
zig build test                         # 55 tests, no window and no speakers needed
zig build run -Doptimize=ReleaseFast   # play it
zig build sounds                       # write the sound effects to zig-out/sounds as .wav
zig build web -Doptimize=ReleaseFast   # browser build; needs $EMSDK or -Demsdk=
zig build -Dframes=600 -Ddemo=play -Dshot=out.png   # headless smoke run
```

Other scripts:

```bash
./update-zig.sh             # refresh toolchain then verify (--check, --force, --version X)
./gh-deploy.sh              # full gate then publish web/dist to gh-pages (--dry-run, --skip-e2e)
```

## Architecture

**The pipeline.** `snippets/*.zig` → `zig build verify` (compile, run under Node's `node:wasi`, diff stdout) → `zig build` installs the wasm + `snippets.json` manifest into `web/public/wasm/` → Astro reads the manifest at build time → the browser fetches the wasm and runs it through `@bjorn3/browser_wasi_shim`. Two runners, one WASI interface: [tools/run-wasi.mjs](tools/run-wasi.mjs) (CI) and [web/src/scripts/wasi-runner.ts](web/src/scripts/wasi-runner.ts) (browser).

**[build.zig](build.zig) is the gate.** It scans `snippets/` at configure time and classifies each file with no configuration:

- contains `pub fn main` → executable, stdout captured; otherwise → `zig test` binary
- sibling `<name>.expected` file → exact-stdout assertion
- `_`-prefixed filename → helper module, not a snippet
- `//! norun` → compiled but never run (panics, filesystem — still gates API drift)
- `//! native` → built and run for the **host** instead of wasm (threads: `wasm32-wasi` is single-threaded)
- `//! norun` **on a native snippet** → compiled and linked for the host, never run. This is what lets the X11 chapter link against the real libX11 (so translate-c drift fails the build) without CI sitting through a window waiting for a keypress.
- `//! link: <lib>` / `//! cinclude: <header>` / `//! csource: <file.c>` → the three ways a snippet involves C, any one of which implies libc and a host build. `link:` names a system library, `cinclude:` names the headers translate-c reads (defaulting to `<lib>.h`, which is why `z` has to say `zlib.h`), and `csource:` names a `.c` file sitting beside the snippet in its chapter directory, compiled into it. The snippet's own directory is on the translate step's include path, so a vendored `calc.h` resolves as `#include <calc.h>`. `csource:` is what lets the Importing C chapter gate the shape it teaches without any library being installed on the runner: Zig ships a C compiler, so the snippet vendors the C it calls.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rajeshpillai/zig-guide](https://github.com/rajeshpillai/zig-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
