---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code plugin that provides GPT (via Codex CLI), Gemini 3 (via the Antigravity CLI `agy`), Grok (via the xAI HTTP API), and OpenRouter (config-driven, advisory-only, 400+ models) as specialized expert subagents. Seven domain experts that can advise OR implement: Architect, Plan Reviewer, Scope Analyst, Code Reviewer, Security Analyst, Researcher, and Debugger. (Grok and OpenRouter are advisory-only - they cannot edit files. Grok reads attached files via the xAI Files API; OpenRouter inlines text files only.)

## Development Commands

```bash
# Test plugin locally (loads from working directory)
claude --plugin-dir /path/to/deliberation

# Run setup to test installation flow
/deliberation:setup

# Run uninstall to test removal flow
/deliberation:uninstall
```

No build step, no dependencies. Codex exposes a native MCP server; Gemini, Grok, and OpenRouter use bundled zero-dependency Node bridges (`server/gemini/index.js`, `server/grok/index.js`, `server/openrouter/index.js`). The Gemini bridge wraps the Antigravity CLI (`agy`) in print mode. The OpenRouter bridge calls any OpenAI-compatible `/chat/completions` endpoint.

## Architecture

### Repository layout

- **`core/`** - host-neutral, zero runtime deps, strict-typed. Provider interface +
  `toErrorResult` + the opinion schema/envelope (`types.js` / `provider.js`): `OPINION_SCHEMA`
  (`recommendation` + `confidence` enum + optional `dissent_points`/`assumptions`/`tradeoffs`
  `string[]`), `parseOpinion(text) -> OpinionEnvelope` (best-effort, never throws; `structured` =
  parse provenance), advisory `validateOpinion` (`{valid, wellFormed, warnings}`), `OPINION_INSTRUCTIONS`,
  and `parseReview(text) -> {verdict, criticalIssues}` (best-effort, never-throws: fenced-code-skipped
  verdict ladder - `VERDICT:` sentinel / same-line keyword / `Verdict` heading-split / bare token - plus
  the closed 6-category taxonomy with next-line continuation-join) used by the convergence loop. `registry.js` (`selectForAskAll` /
  `selectForConsensus`); `orchestrate.js` (`askAll` / `askOne` / `consensus` / `runToConvergence` -
  the non-Claude server-side loop driver); `consensus-loop.js` (the PURE convergence state machine -
  the SSOT for round counting, the convergence rule, the configurable max-rounds cap, history, and the
  confidence label); `loop-store.js` (ephemeral sliding-TTL + LRU `Map` holding `LoopState` across the
  stateless `consensus-step` calls; independent of `sessions.persist`); `providers/*.js`
  adapters (`codex.js` spawns the Codex CLI; `antigravity.js` / `grok.js` /
  `openai-compatible.js` wrap their bridge via an injectable `opts.bridge`); `paths.js`
  (config + cache path resolver, `DELIBERATION_CONFIG` override).
- **`server/mcp/`** - stdio JSON-RPC MCP server over `core`. Published as
  `@antonbabenko/deliberation-mcp`: an esbuild `prepack` step bundles `core` + the three bridges
  into a self-contained `dist/index.js` (build-time devDep only; `dist/` gitignored). `server.json`
  at the repo root is the Official MCP Registry manifest.
- **`server/{gemini,grok,openrouter}/`** - the provider bridges (gemini wraps the `agy` CLI;
  grok = xAI HTTP; openrouter = any OpenAI-compatible HTTP) plus openrouter `config.js`
  (`validateConfig` / `makeConfigReader`, the config SSOT). Registered (with the unified
  `server/mcp` server) inline in `.claude-plugin/plugin.json` under the `mcpServers` key
  (`deliberation-*` / `deliberation`) - this inline block is the SOLE runtime MCP registration.
  Claude Code reads MCP servers from a plugin's root `.mcp.json` OR inline in `plugin.json`; the
  inline form is used so the manifest is NOT also auto-loaded as a project-scope `.mcp.json` when
  working inside this repo (which would duplicate every server with an unresolved
  `${CLAUDE_PLUGIN_ROOT}`). The args use `${CLAUDE_PLUGIN_ROOT}`, which Claude Code resolves to the
  installed version on every load, so updating is just `/plugin marketplace update antonbabenko` + `/reload-plugins`. `/deliberation:setup` seeds config and installs rules; it does not register MCP servers.
- **Typecheck gate** - `tsconfig.json` strict `checkJs` over `core/**` + `server/mcp/**/*.js`
  (excludes `server/mcp/dist`). `npm run check` = `typecheck` + `node --test test/*.test.js`,
  enforced in CI by `.github/workflows/validate.yml`.

### Consensus engine (single source of truth)

The multi-round convergence loop lives in `core/consensus-loop.js` as a pure state machine
(`init -> await_blind -> await_peers -> await_adjudication -> converged|await_revision -> ...`),
shared by two drivers so there is ONE rules layer, not a Claude copy and a non-Claude copy:

- **`consensus`** (MCP tool) - runs the whole loop server-side in one call with a CONCRETE
  provider arbiter (`core/orchestrate.js runToConvergence`). `maxRounds` overrides the config cap;
  `synthesizeAlways:true` runs a SINGLE arbiter synthesis pass instead of the loop (free-text
  `synthesis`, for open questions) - one unified tool, one return envelope (split `verdict`/`synthesis`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antonbabenko/deliberation](https://github.com/antonbabenko/deliberation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
