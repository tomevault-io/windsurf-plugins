---
trigger: always_on
description: Static analysis tool that maps API endpoints and detects client-side security issues by analyzing Next.js (webpack/turbopack) and Vue.js bundles. Written in TypeScript, compiled to `build/` before running.
---

# js-recon

Static analysis tool that maps API endpoints and detects client-side security issues by analyzing Next.js (webpack/turbopack) and Vue.js bundles. Written in TypeScript, compiled to `build/` before running.

## STRICT: Repository hygiene

**Research artifacts must never be committed to this repo.** This is a public tool repository. Its git history must contain only tool source code, tests, docs, and configuration. Never commit:

- Experiment scripts, research notes, or analysis results
- Files from `js-recon-research/` or any private workspace directory
- Prompt logs, observation markdown files, or scratch files

Research outputs belong in the private sibling workspace outside this repo. If an experiment script or results file is needed as reference, keep it in the private workspace only.

## Build & run

```bash
npm run cleanup   # rm -rf build/ + tsc (full rebuild)
npm run start -- <subcommand> [options]
```

`cleanup` must be run before testing any TypeScript change when using the `run` command.

## Subcommands

| Command       | Purpose                                                                            |
| ------------- | ---------------------------------------------------------------------------------- |
| `lazyload`    | Download JS chunks from a target URL                                               |
| `strings`     | Extract strings/paths/secrets from JS files                                        |
| `map`         | Parse webpack/turbopack bundles into a structured `mapped.json`                    |
| `endpoints`   | Extract client-side routes                                                         |
| `analyze`     | Run YAML rules against `mapped.json` / OpenAPI spec                                |
| `report`      | Generate HTML/SQLite report                                                        |
| `run`         | Run all of the above in sequence (primary interface)                               |
| `api-gateway` | Manage AWS API Gateway for IP rotation                                             |
| `mcp`         | AI-powered CLI / one-shot chat (`-c`) / Model Context Protocol server (`--server`) |
| `cs-mast`     | Compute CS-MAST structural hashes for downloaded JS files; find hash collisions    |
| `sourcemaps`  | Extract source files from `.map` sourcemap file(s)                                 |

## Key source files

- `src/index.ts` — CLI entry point; all subcommand definitions and option declarations live here
- `src/run/index.ts` — orchestrates the full pipeline (`run` subcommand); two tech-specific flows (Next.js 8-step, Vue 4-step)
- `src/analyze/index.ts` — loads/validates rules, runs AST and request engines
- `src/analyze/helpers/initRules.ts` — downloads/caches rules from GitHub to `~/.js-recon/rules`
- `src/analyze/helpers/validate.ts` — validates rules and checks `js_recon_version` compatibility
- `src/analyze/helpers/schemas.ts` — Zod schema for rule YAML files
- `src/map/graphql/resolveGraphql.ts` — framework-agnostic GraphQL operation scanner. Visits every `StringLiteral` and `TemplateLiteral` in every JS file, validates with the `graphql` library's `parse()`, and emits each operation as a POST request under a flat `GraphQL` collection folder. Inlines transitively-referenced fragment definitions into each printed query so emitted requests are self-contained. Runs in every framework branch of `map/index.ts` when `--openapi` is on and `--no-graphql`/`--ngql` is not set.
- `src/map/next_js/resolveFetch.ts` — resolves `fetch()` calls, detects Next.js framework chunks
- `src/map/next_js/resolveServerActions.ts` — detects `createServerReference(actionId, ...)` calls, derives App Router routes from chunk file paths, traces argument call sites (same-chunk and cross-chunk), and emits POST endpoints with `next-action` headers and typed arg hints (e.g. `<string:userId>`) into the global OpenAPI output
- `src/map/next_js/utils.ts` — `resolveNodeValue`, `resolveVariableInChunk`, `substituteVariablesInString`
- `src/map/vue_js/vue_resolveXhr.ts` — directory-scan resolver for `new XMLHttpRequest()` + `.open()/.setRequestHeader()/.send()` patterns. Shared by Vue/React/Svelte pipelines; the `frameworkName` arg only changes log labels. Reaches ground-truth XHR sites but in axios/Got/Ky-style bundles the URL/method come from a dispatcher config (`re.url`, `re.method`) and resolve only to opaque `[member:re.url]` placeholders that taint analysis cannot unwind across the library's internal dispatch chain — those entries fail the `looksLikeUrl` check at emit time. Catch the wrapper-level call instead via `vue_resolveHttpClient`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [js-recon/js-recon](https://github.com/js-recon/js-recon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
