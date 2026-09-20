---
trigger: always_on
description: threeforge 0.9.2 is a frame-budget compiler and diagnostics layer for three.js games (r186, WebGPU with a
---

# threeforge for AI agents

threeforge 0.9.2 is a frame-budget compiler and diagnostics layer for three.js games (r186, WebGPU with a
WebGL2 fallback). This file is what `npx threeforge` prints. Everything below is scriptable from a terminal:
`analyze`, `inspect`, `optimize` and `explain` print JSON with `--json`, `schema` prints JSON either way, and
`mcp`, `decoders` and `help` take no `--json`.

## Install

```bash
npm i -D threeforge playwright && npx playwright install chromium
```

Playwright is only needed for `analyze`, `inspect`, `optimize` (its verification) and `mcp`; the library itself has no
such dependency. `optimize` works out of the box (glTF-Transform is a dependency); texture compression needs
`npm i -D sharp` and a Draco-compressed input needs `npm i -D draco3dgltf`.

## Commands

| command | what it does |
|---|---|
| `npx threeforge analyze <file.glb\|.gltf> [--backend webgl2\|webgpu] [--tier auto\|desktop\|phone-mid\|phone-low] [--budget N] [--frames N] [--no-compile] [--timeout ms] [--headed] [--bake] [--bake-buried] [--views N] [--parity pct] [--json]` | Renders the asset headlessly, measures every cost category, compiles (batches, or bakes with `--bake`) it, measures again, checks pixel parity from the default framing plus `--views` orbit views, returns hints and a verdict. |
| `npx threeforge inspect <url> [--backend webgl2\|webgpu] [--budget N] [--frames N] [--no-compile] [--timeout ms] [--headed] [--json]` | Drives your running app (dev server) through `window.__threeforge`, compiling through the hook unless `--no-compile`; same document without asset facts and parity. The app measures itself at the tier its ledger detects, so there is no `tier` flag here. |
| `npx threeforge optimize <file.glb\|.gltf> [--out out.glb] [--preset safe\|balanced\|aggressive] [--no-<step>\|--<step>] [--simplify [ratio]] [--simplify-error e] [--compress none\|meshopt] [--textures [webp\|avif\|none]] [--texture-size N] [--texture-quality Q] [--no-verify] [--parity pct] [--views N] [--budget N] [--backend webgl2\|webgpu] [--tier auto\|desktop\|phone-mid\|phone-low] [--frames N] [--no-compile] [--timeout ms] [--headed] [--json]` | Rewrites the asset with glTF-Transform and writes `<name>.forge.glb`. `safe` (default) is dedup, palette, prune, measured at 0 changed pixels (no channel moving by more than 24 of 255) on the Fox and the Buggy; `palette` adds a UV attribute to every primitive whose flat materials it merges, so it can make a file bigger. `balanced` adds weld, resample, quantize and WebP textures (2048 px); `aggressive` adds simplify to 50 % and 1024 px textures. Renders the original and the result, compares pixels, compiles both, and lists what the file needs at load time (`requires`). |
| `npx threeforge explain [<hint-code>] [--all] [--json]` | What a hint means, what to change, which API (a hint code or `--all`, not both). |
| `npx threeforge schema [snapshot\|analyze\|inspect\|optimize\|all] [--json]` | JSON Schemas (draft 2020-12) of everything the commands print. |
| `npx threeforge mcp` | Stdio MCP server with tools `analyze_asset`, `inspect_app`, `optimize_asset`, `explain_hint` (needs `npm i -D @modelcontextprotocol/sdk zod`). |
| `npx threeforge decoders <dir>` | Copies three's Draco decoder and Basis transcoder into `<dir>/draco` and `<dir>/basis` for `createLoader(renderer, { decoders })`. No JSON output. |

Exit codes: `0` pass · `1` verdict failed (over budget, an error-severity hint, pixel parity lost, or a page error during
`analyze`/`optimize`) · `2` usage or
input error · `3` environment (Playwright or Chromium missing; the message has the install command) · `4` the page
threw or timed out. In `--json` mode stdout is only the JSON document; the human summary goes to stderr.

## Flags

Flags follow the command, before or after its argument. A value is `--flag value` or `--flag=value`; boolean flags
never take one, so `analyze --json scene.glb` works. `--simplify` and `--textures` take a value only after `=` or
when the next argument is a valid value. `--` ends the flags (for a path that starts with `-`). `--help` on any
command prints this file. An unknown flag (the message suggests the nearest one), an extra argument, a flag given twice,
a malformed or out-of-range number, `inspect --tier` and `optimize --budget` with `--no-verify` exit `2` with
nothing on stdout.

| flag | commands | meaning |
|---|---|---|
| `--backend webgl2\|webgpu` | analyze, inspect, optimize | Renderer backend to measure on (default `webgl2`). |
| `--tier auto\|desktop\|phone-mid\|phone-low` | analyze, optimize | Device tier for budgets and hints (default `auto`: detected from the GPU and device). |
| `--budget N` | analyze, inspect | Fail the verdict (exit 1) above N scene submissions after compiling (an integer ≥ 0). |
| `--frames N` | analyze, inspect, optimize | Frames to measure; costs are medians (an integer ≥ 1, default 30). |
| `--compile`, `--no-compile` | analyze, inspect, optimize | Compile (batch) the scene and measure again. On by default; `--no-compile` measures the scene as loaded. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tallslab/threeforge](https://github.com/tallslab/threeforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
