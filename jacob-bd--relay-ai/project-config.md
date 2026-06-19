---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

```bash
npm run build       # compile TypeScript → dist/cli.js (via tsup, ESM, shebang injected)
npm test            # run all tests with vitest
npm run typecheck   # type-check without emitting (tsc --noEmit)
npm run dev         # watch mode build

# Run a single test file
npx vitest run tests/env.test.ts
npx vitest run tests/models.test.ts

# Test the CLI locally (already npm-linked)
relay-ai --help
relay-ai models          # manage favorite models for mid-session switching
relay-ai Codex --dry-run   # simulate full first-run without writing anything
relay-ai Codex --setup    # re-ask subscription tier
relay-ai Codex --trace    # write debug log to /tmp/relay-ai-debug.log and print errors on exit
relay-ai server           # foreground OpenCode/registry API gateway
relay-ai server --vertex  # foreground Vertex AI gateway (gcloud ADC)
relay-ai codex            # Codex CLI with registry providers (see docs/CODEX.md)
relay-ai codex-app        # Codex desktop app (macOS/Windows; see docs/CODEX.md)

# Rebuild after code changes before testing manually
npm run build && relay-ai --version
```

## Architecture

**Entry point:** `src/cli.ts` orchestrates the full flow. Every other module is a focused unit with no side effects at import time.

**Data flow (`relay-ai Codex`):**
```
cli.ts
  → findClaudeBinary()         [launch.ts — locate Codex binary]
  → fetchLocalProviders()      [providers.ts — ephemeral opencode serve, GET /config/providers, normalize]
  → p.select "Which provider?" [shown when local providers are available]

  ── OpenCode cloud path (default) ──
  → resolveOrCollectApiKey()   [reads env, OS credential store (all platforms), or prompts user]
  → askSubscriptionTier()      [prompts.ts — one-time question, saved to conf store]
  → getModels()                [models.ts — API fetch + cache enrichment + format classification]
  → runWizard()                [prompts.ts — backend/model selector, filters unsupported]

  ── Local provider path ──
  → pickLocalModel()           [prompts.ts — filter/select model from local provider]

  ── Shared launch (no favorites) ──
  → startProxy()               [proxy.ts — single-model wrapper around startProxyCatalog]
  → buildChildEnv(baseUrl, …)  [env.ts — removes 17 conflicting vars, sets OpenCode vars]
  → launchClaude()             [launch.ts — spawn with stdio:inherit]
  → proxyHandle.close()        [stops proxy after Codex exits]

  ── Switch-menu launch (favorites.length > 0) ──
  → buildCatalogRoutes()       [catalog.ts — starting model + favorites, max 20]
  → startProxyCatalog()        [proxy.ts — multi-route proxy, alias IDs per model]
  → buildChildEnv(…, gatewayDiscovery=true)  [sets CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1]
  → launchClaudeViaCatalog()   [cli.ts — shared launch + trace cleanup]
```

**`relay-ai models`:** Interactive favorites manager (`src/favorites.ts`). Reads/writes `favoriteModels` in config. Saves once on Done. Stale favorites (unavailable models) are silently skipped when building the catalog.

**Catalog routing** (`src/catalog.ts`): `localModelToRoute`, `zenGoModelToRoute`, `makeRouteResolver`, `buildCatalogRoutes`. Routes built only for starting model + favorites — not the full model list. Alias IDs via `aliasModelId()` in proxy so Codex sees unique model names in `/model`.

**Critical URL constraint:** `BACKENDS.baseUrl` in `constants.ts` must NOT include `/v1`. The Anthropic SDK appends `/v1/messages` automatically. Setting it to `https://opencode.ai/zen/v1` would cause requests to hit `/zen/v1/v1/messages` → 404.

**Model discovery two-source merge:**
- Primary: `GET {backendUrl}/v1/models` (no auth needed, returns available IDs)
- Enrichment: `~/.cache/opencode/models.json` (written by OpenCode CLI) — provides `name`, `family`, `cost`, `provider.npm`
- `isAnthropicNative`: true when `modelFormat === 'anthropic'`
- `modelFormat`: classified from `provider.npm` in cache, or by ID-prefix heuristic:
  - `@ai-sdk/anthropic` or `Codex-*` → `'anthropic'` (direct passthrough)
  - `@ai-sdk/openai` or `gpt-*` → `'unsupported'` in the **cloud OpenCode wizard** (OpenCode Zen/Go proxy layer; not direct OpenAI). Use the **local OpenAI provider** instead for GPT models.
  - `@ai-sdk/google` or `gemini-*` → `'unsupported'` (needs model-specific endpoints)
  - Everything else → `'openai'` (routed through the SDK adapter via the local proxy)
- `sourceBackend`: set from the backend that was queried — critical for `go` tier which shows Zen free models + Go paid models in one list, so the correct `ANTHROPIC_BASE_URL` can be set per selected model

**Translation layer — the Vercel AI SDK adapter** (`src/sdk-adapter.ts` + `src/provider-factory.ts`): All non-Anthropic providers route through the Vercel AI SDK (`ai` + `@ai-sdk/*`, the same packages OpenCode loads), which owns wire format, endpoint selection, and provider quirks. This is the **single** translation path — there is no hand-rolled per-provider translation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacob-bd/relay-ai](https://github.com/jacob-bd/relay-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
