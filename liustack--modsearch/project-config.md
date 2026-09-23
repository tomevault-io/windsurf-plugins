---
trigger: always_on
description: Provide the `modsearch` CLI tool that turns search queries and page URLs into structured web evidence for LLM agent workflows.
---

# Project Overview (for AI Agent)

## Goal

Provide the `modsearch` CLI tool that turns search queries and page URLs into structured web evidence for LLM agent workflows.

## Technical Approach

- **Pluggable engines** — the engine interface (`buildInvocation` + `parseOutput` for subprocess engines, or `execute` for in-process ones) keeps each engine contained to one file, so adding or swapping one touches nothing else. Antigravity CLI (`agy`) is the preferred search engine when it is available.
- **Two modes, one CLI**: `-q` searches the web, `-u` fetches a single page (absorbed from the retired `modfetch` project). `-u` plus `-q` fetches with an answer focus.
- **Schema-enforced JSON output**: subprocess engines are invoked with `--json-schema`, so the structured result comes back guaranteed, no markdown scraping. grok-cli is the exception: that flag disables its search loop, so the result is salvaged from the final text.
- **Selectable automatic chain**: every engine participates by default. `engines.<name>.enabled: false` excludes one from automatic search, fetch, or X routing. An explicit `--engine` still forces a one-off run. `doctor` reports readiness and enabled state separately.
- **Hidden Windows children**: the core and standalone dsh plugin each own one child-process wrapper that forces `windowsHide: true`. A contract test rejects direct `child_process` access anywhere else in shipped code.
- **Single responsibility**: this project handles the live web (search + fetch). Image parsing lives in `modlens`.
- **X handling**: X-flavored `-q` queries route entirely to Grok Build (`grok-cli`) when it is installed and signed in, spending no agy quota. When Grok is missing or fails, a web engine answers instead, and that entry is marked `status: "degraded"` with `source: "web"` and `requestedSource: "x"` so the second-hand nature is explicit, never silent. On a `--source web,x` run with X unreachable, the X slot returns a separate `status: "unavailable"` entry rather than vanishing. An explicit `-e`/`--engine` is a hard force: exactly that engine, no fallback, an error if it cannot do the job. One shared output contract for every engine.

```bash
pnpm install
```

## Code Organization

```
src/
├── main.ts       # CLI entry: search/fetch command + doctor + config subcommands
├── search.ts     # orchestration: mode, plan, run each source concurrently, envelope
├── doctor.ts     # `modsearch doctor`: diagnose config + routing, no quota, no network
├── router.ts     # every routing decision: sources, role to engine, fallbacks
├── config.ts     # layered engine config: flags > env > ~/.modsearch/config.json
├── subprocess.ts # running an engine binary and draining its output
├── util/
│   └── spawnHidden.ts # the only core child-process boundary
├── system.ts     # tiny host helpers (is this binary on PATH)
├── prompt.ts     # search + fetch prompt templates
├── schema.ts     # JSON schemas enforced on engines that support them
└── providers/
    ├── index.ts        # engine interface + registry + role preference
    ├── antigravity.ts  # agy: search + fetch
    ├── tavily.ts       # Tavily API: search
    ├── exa.ts          # Exa API: search
    ├── firecrawl.ts    # Firecrawl API: search + fetch
    ├── grok.ts         # Grok Build: X
    └── httpFetch.ts    # direct HTTP fetch, no dependencies

Tests are co-located: each module has an adjacent `*.test.ts`.
```

## Skills Directory

```
skills/
└── modsearch/
    ├── SKILL.md
    └── references/
        ├── configure.md
        └── output-schema.md
```

The CLI is exposed via `dist/main.js`.

## dsh Plugin (`dsh/`)

`dsh/index.js` is a DeepSeek Harness plugin: plain JS, node builtins only, no build step, no dsh package imports. It spawns the CLI from `dist/main.js` inside the same package and plugs in three ways: the engine chain becomes the web seam's search provider (`cordis.patch.yml` repoints the seam's `searchProvider` at it, so dsh's native `web_search` tool runs on modsearch), and `x_search` / `read_page` register as raw tools for the two capabilities dsh has no seam for. The schema copies `dsh/search-schema.json` and `dsh/fetch-schema.json` are kept in lockstep with `src/schema.ts` by `src/dshPlugin.test.ts`, which also tests the plugin's behavior against a fake CLI via the `MODSEARCH_DSH_CLI` env override. The package.json `dsh.bundle` manifest plus the root export make the package installable with `dsh plugin add`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liustack/modsearch](https://github.com/liustack/modsearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
