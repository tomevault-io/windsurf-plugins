---
trigger: always_on
description: This file provides guidance to AI coding agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents working with code in this repository.

PR policy: AI-generated PRs are welcome and will be reviewed and accepted if the diff is clean, tested, and follows the conventions below. Tag the PR with `ai-authored` so reviewers can apply the right scrutiny.

See also `../luau2ts.com/` for the docs site and playground that consume this package.

## Project Overview

`luau2ts` is a Luau-to-TypeScript compiler. It reads Luau source (the Roblox dialect of Lua) and emits TypeScript, optionally targeting the [`roblox-ts`](https://roblox-ts.com) ecosystem so the output drops back into a Roblox build via `rbxtsc`. The package ships as a CLI (`luau2ts`) and as a library (`compile(source, options)`), and also produces `.d.ts` declaration files alongside compiled output for migration use cases.

The repo is a pnpm workspace with one optional sub-package (`@luau2ts/analyzer`, a WASM build of the official Luau analyzer used for the optional `--check-luau` pass).

## Development Commands

### Building
```bash
pnpm build                                 # Build the root compiler (runs prebuild then `tsc -b`)
pnpm --filter @luau2ts/analyzer build      # Build the analyzer WASM (one-time, slow)
```

`prebuild` runs two code generators:
- `scripts/build-oracle.mjs` vendors `@rbxts/types` data into `src/compile/oracle/data.generated.ts`.
- `scripts/build-api-macros.mjs` generates api-data, exclusions, stdlib-slots, datatype-slots.

If you touch the generators or `@rbxts/types`, rerun `pnpm build` to refresh `data.generated.ts`.

### Tests
```bash
pnpm test                                  # Run the vitest suite (~295 tests)
pnpm test:watch                            # Watch mode
pnpm typecheck                             # tsc --noEmit
```

The suite is in `src/compile.test.ts` plus a few smaller files. Most tests are golden-file: a Luau snippet in, a TS string expected out. The test runner snapshots exact emit (including helper imports), so a change in cast emission or pretty-printing forces test updates.

### Stress harness
```bash
node scripts/stress-rbxl.mjs <path-to-rbxl> [--mode rbxts|native] [--dump <outDir>] [--limit N] [--verbose]
```

Compiles every script inside an `.rbxl` Roblox place file. Reports parse / TS / Luau errors per script, lists worst offenders, and optionally dumps the compiled `.ts` tree (and `.d.ts` sidecar) to `<outDir>` for inspection with `rbxtsc` or hand-written TS consumers. The canonical stress corpus is `C:/Users/tonyt/Desktop/thisisatest.rbxl` (127 scripts). The harness uses the sibling repo `rbx-web`'s rbxl binary parser.

### Roundtrip
The end-to-end smoke test lives in the sibling `test2/` directory (`pnpm link`ed to this repo):
```bash
cd ../test2
node roundtrip.mjs [<fixture.luau>]        # Simple.luau by default
node roundtrip-corpus.mjs                  # Roundtrip every dumped corpus script
```

`roundtrip.mjs` runs Luau through luau2ts, then through roblox-ts (`rbxtsc --type package`) back to Lua, then StyLua-formats both ends so a diff highlights real semantic differences. Exit 0 means clean.

### Canary
```bash
node scripts/canary.mjs                    # One specific Luau snippet, prints the emitted TS
```

A hand-curated input that exercises chain-of-method-calls, multi-return, and string-interpolation patterns. Used as a fast eye-check after compiler changes.

## Architecture Overview

### Project Structure
- `src/parser/`: Luau parser (WASM-backed by the official Luau parser, see `wasm/`).
- `src/compile/`: the compiler itself, around 9000 lines of TypeScript in `index.ts` plus several pre-passes.
- `src/compile/oracle/`: vendored `@rbxts/types` lookup tables: class hierarchy, method/property signatures, conventional child-name resolution.
- `src/compile/macros/`: per-namespace API macros (`math`, `string`, `table`, `Instance`, datatype ctors).
- `src/compile/cross-script/`: corpus-aware analysis. Builds a per-module index, emits `.d.ts` files, collects cross-script call sites for function-param backprop.
- `src/cli/`: CLI entry (`bin.ts`), arg parser (`args.ts`), output modes (`modes.ts`).
- `src/rojo/`: Rojo project-file walker. Resolves `default.project.json` to a flat script list with Roblox instance paths.
- `src/runtime/`: helper functions imported by `native` mode emit (`luaIndex`, `lualen`, `pairKeys`, `multiret`, `truthy`, etc).
- `packages/analyzer/`: optional WASM build of the official Luau analyzer for `--check-luau`.
- `scripts/`: build-time generators, stress harness, canary.
- `test2/` (sibling): roundtrip fixture and end-to-end stage.

### Key Entry Points
- `src/compile/index.ts:compile(source, options)` is the library entry. Parses, runs pre-passes, emits TS source string plus errors.
- `src/cli/bin.ts` is the CLI entry, dispatches to one of three modes.
- `src/cli/modes.ts` defines `compileFileMode`, `compileDirMode`, `compileProjectMode` for single file, directory tree, and Rojo project respectively.

### Compile Pipeline
A call to `compile(source, options)` walks this sequence (rbxts mode shown; native mode skips some passes):

1. **Parse**: Luau AST via the WASM parser.
2. **Pre-passes** (rbxts only):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Luau2TS/luau2ts](https://github.com/Luau2TS/luau2ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
