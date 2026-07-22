---
trigger: always_on
description: > This file helps AI agents understand the codebase quickly. Read it before making changes.
---

# pi-free — Agents.md

> This file helps AI agents understand the codebase quickly. Read it before making changes.

## What is pi-free?

A **Pi extension** (`@earendil-works/pi-coding-agent`) that registers free and paid AI model providers with Pi's model picker. It shows free models by default and lets users toggle per-provider between free-only and all-models view via `/toggle-{provider}` commands.

**Package:** `pi-free` v2.2.7
**Author:** Apostolos Mantzaris  
**License:** MIT  
**Repo:** `github.com/apmantza/pi-free`  
**Peer deps:** `@earendil-works/pi-ai`, `@earendil-works/pi-coding-agent`, `@earendil-works/pi-tui`

---

## Architecture at a Glance

```
index.ts                          ← Extension entry point (piFreeEntry)
  ├─ lib/registry.ts              ← Global provider registry + isFreeModel detection
  ├─ lib/toggle-state.ts          ← Generic toggle state machine (free ↔ all)
  ├─ lib/built-in-toggle.ts       ← Toggles for Pi's built-in providers (opencode, openrouter)
  ├─ lib/quota-monitor.ts         ← Rate-limit header extraction → status bar
  ├─ lib/logger.ts                ← Structured logging (console + ~/.pi/free.log)
  ├─ lib/json-persistence.ts      ← Generic JSON/JSONL file stores
  ├─ lib/model-detection.ts       ← Model family grouping, name normalization
  ├─ lib/model-enhancer.ts        ← CI score name decoration (thin wrapper)
  ├─ lib/provider-cache.ts        ← Disk cache for fetched model lists
  ├─ lib/provider-compat.ts       ← DeepSeek proxy compat flag detection
  ├─ lib/util.ts                  ← fetchWithRetry, model size parsing, OpenRouter mapping
  │
  ├─ config.ts                    ← ~/.pi/free.json + env var resolution (ALL config lives here)
  ├─ constants.ts                 ← Provider IDs, base URLs, timeouts, thresholds
  ├─ provider-helper.ts           ← registerOpenAICompatible, createReRegister, enhanceWithCI, setupProvider
  │
  ├─ provider-failover/           ← Benchmark lookup (Coding Index scores)
  │   ├─ benchmark-lookup.ts      ← Multi-strategy benchmark matching + debug logging
  │   ├─ hardcoded-benchmarks.ts  ← Benchmark data
  │   └─ benchmarks-chunk-*.ts    ← Split benchmark data files
  │
  └─ providers/                   ← Per-provider extensions (each exports default async fn)
      ├─ kilo/kilo.ts             ← Kilo Gateway (OAuth, free + paid)
      ├─ cline/cline.ts           ← Cline bot (OAuth, message reshaping for Cline API)
      ├─ novita/novita.ts         ← Novita AI (paid credits)
      ├─ ollama/ollama.ts         ← Ollama Cloud (usage-based free tier, 403 probing)
      ├─ routeway/routeway.ts     ← RouteWay AI (paid)
      ├─ sambanova/sambanova.ts   ← SambaNova (free tier)
      ├─ zenmux/zenmux.ts         ← ZenMux AI gateway (paid)
      ├─ crofai/crofai.ts         ← CrofAI (paid)
      ├─ llm7/llm7.ts             ← LLM7 (free default/fast selectors)
      ├─ deepinfra/deepinfra.ts   ← DeepInfra ($5 trial credit)
      ├─ together/together.ts     ← Together AI (paid credits)
      ├─ tokenrouter/tokenrouter.ts ← TokenRouter API gateway (paid + free models)
      ├─ anyapi/anyapi.ts         ← AnyAPI gateway (free plan + free models)
      ├─ model-fetcher.ts         ← Shared OpenRouter-compatible model fetching
      ├─ opencode-session.ts      ← OpenCode session handling
      └─ dynamic-built-in/        ← Dynamic fetchers for Mistral, Groq, Cerebras, xAI, HF
          └─ index.ts

tests/                            ← Vitest test suite
```

---

## Key Concepts

### Extension Entry Point

`index.ts` exports `piFreeEntry(pi: ExtensionAPI)` — the single entry point Pi calls. It:

1. Sets up global commands (`/toggle-free`, `/free-providers`)
2. Sets up quota monitoring (passive, listens to `after_provider_response`)
3. Loads all unique providers via `Promise.allSettled`
4. Sets up dynamic built-in providers (only if API keys configured)
5. Sets up built-in provider toggles (OpenCode, OpenRouter)
6. Applies initial global filter if `free_only` is enabled

### Provider Registration Pattern

Every provider follows this pattern:

```typescript
export default async function providerName(pi: ExtensionAPI) {
    // 1. Fetch models (from API, hardcoded list, or models.dev)
    const allModels = await fetchModels(...);
    const freeModels = allModels.filter(m => isFreeModel(m, allModels));
    const stored = { free: freeModels, all: allModels };

    // 2. Create re-register function (used by toggles)
    const reRegister = createReRegister(pi, { providerId, baseUrl, apiKey });

    // 3. Register with global toggle system
    registerWithGlobalToggle(providerId, stored, reRegister, hasKey);

    // 4. Register initial models with Pi
    pi.registerProvider(providerId, { models: enhanceWithCI(initialModels), ... });

    // 5. Register toggle command
    pi.registerCommand(`toggle-${providerId}`, { ... });

    // 6. Status bar + session refresh
    pi.on("model_select", ...);
    pi.on("session_start", ...);
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apmantza/pi-free](https://github.com/apmantza/pi-free) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
