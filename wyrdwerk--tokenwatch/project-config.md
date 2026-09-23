---
trigger: always_on
description: Before ANY tool call on a new task, STOP and classify:
---

# CRITICAL: Delegation Gate

Before ANY tool call on a new task, STOP and classify:
1. Will this task require reading 3+ files or running multiple analysis scripts?
   → YES: Dispatch `explore` subagent(s) FIRST. Do NOT read files inline.
2. Is this a code edit that depends on investigation findings?
   → Investigate via subagent, then edit inline with the returned findings.
3. Is this a single quick lookup or sequential edit?
   → Proceed inline.

Rules: Investigation = delegate. Editing/deciding = inline. Never both.

---

# AGENTS.md — TokenWatch

## Project overview

Static site comparing pay-as-you-go LLM API pricing across inference providers. Uses OpenRouter's `/endpoints` API to de-aggregate per-backend pricing (each backend like DeepInfra, Fireworks, Together becomes its own row). Zero dependencies, pure Node ESM. Deployed to Cloudflare Pages with 2-hourly CI/CD refresh + auto-deploy on push.

> **Diagram:** [ARCHITECTURE.md](ARCHITECTURE.md) — three-pipeline flowchart (text / image / video).
> **Canonicalization traps:** [docs/canonicalization-edge-cases.md](docs/canonicalization-edge-cases.md) — read before touching `shared/normalize.mjs` or `public/app.js`.
> **Decisions:** [docs/adr/](docs/adr/) — why the pipeline is shaped this way.
> **SEO infrastructure:** `scripts/generate-seo.mjs` server-renders the 25 cheapest models into `index.html` for crawlability, and generates `sitemap.xml` + `robots.txt`. Run `npm run seo` before deploy. See [docs/conversations/20260803-seo-gsc-setup-public.md](docs/conversations/20260803-seo-gsc-setup-public.md).

## Architecture

- **Data pipeline**: `scripts/fetch-pricing.mjs` fetches pricing from 3 tiers:
- **Tier 1 — Direct providers** (authoritative): DeepInfra, Crof, EmberCloud, Wafer, Synthetic, Lilac, SambaNova, HyperCharm, Sference, Neuralwatt, Merius, Aster Labs, SingularityAPI, RunInfra — fetched via their own `/v1/models` endpoints. SingularityAPI and RunInfra require `SINGULARITY_API_KEY` / `RUNINFRA_API_KEY` (skipped with a warning if unset). LLM Gateway (`LLMGATEWAY_API_KEY`) is also Tier 1 but **differential only**: it emits priced text rows for hosts TokenWatch does not already fetch (ByteDance, Runware, SCX, Canopywave, Iceberg, NanoGPT, …). Image-output SKUs and already-covered backends (OpenAI, Azure, Novita, Z.ai, …) are dropped.
- **Tier 2 — OpenRouter de-aggregated**: `/v1/models` lists models, then `/endpoints` per model returns per-backend pricing. Each backend (Fireworks, Together, Novita, SiliconFlow, etc.) becomes its own row — NOT "OpenRouter"
- **Tier 3 — CSV/hardcoded/scraped**: Makora, Xiaomimimo (CSV), OpenCode Go (docs-page scrape: `parseOpenCodeGoDocs()` in `scripts/lib.mjs` scrapes the pricing table at `https://opencode.ai/docs/go/` — the `/zen/go/v1/models` catalog endpoint lists models but has no prices; context lengths stay manual via `OPENCODE_GO_CONTEXT`; legacy hardcoded array is the fallback), Umans (manually maintained `UMANS_MODELS` / `parseUmansHardcoded()` — not a live `/v1/models` fetch; status.umans.ai SSR is for performance data only)
  - **3-tier precedence**: dedup key is `canonicalId(m.id) | normalized_provider` (`scripts/lib.mjs:282-284`). Direct wins over OpenRouter, which wins over CSV/hardcoded; first-seen/highest-tier wins among identical keys. **Quantization IS part of the dedup key** — `canonicalId()` preserves quant suffixes (`shared/normalize.mjs:34-47`), so different quants of the same model+provider produce distinct keys and stay distinct rows (`test/canonicalization.test.mjs:88-97`). (Note: `orgLookupKey` strips quant for org resolution only — `normalize.mjs:54-58` — and is NOT the dedup key; see `docs/canonicalization-edge-cases.md` §2.)
  - Writes `public/pricing.json` with 1,180 text-generation models across 82 inference providers (2026-08-11 data). **~56.4% are ZDR-tagged.**
- **models.dev enrichment**: after the 3-tier fetch + dedup, `fetch-pricing.mjs` calls `fetchModelsDevEnrichment()` (sidecar, non-fatal) which pulls `https://models.dev/api.json` and builds a `(provider, normalizedModelId)` index. `applyEnrichment()` decorates each model with a `modelsdev` block (base URL, native model ID, capability metadata) and fills `null` cache_read/cache_write/context_length/max_output values. Never overwrites existing values. Two-tier matching: Tier A (exact normalized, confidence `'high'`) + Tier B (bounded fuzzy subset, confidence `'medium'`, surfaces a ⚠ pill in the UI).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WyrdWerk/tokenwatch](https://github.com/WyrdWerk/tokenwatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
