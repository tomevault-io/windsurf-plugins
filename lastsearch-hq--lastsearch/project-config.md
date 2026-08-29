---
trigger: always_on
description: LastSearch is open-source research infrastructure for AI agents. It gives agents real-time web search with evidence-backed citations and confidence scores. Available as MCP server, REST API, and Python SDK.
---

# CLAUDE.md — LastSearch

## What this project is

LastSearch is open-source research infrastructure for AI agents. It gives agents real-time web search with evidence-backed citations and confidence scores. Available as MCP server, REST API, and Python SDK.

**Tagline:** Reliable Research Infrastructure for AI Agents

## Monorepo structure

```
apps/mcp/          — MCP server (npm: lastsearch)
packages/shared/   — Shared types, schemas, constants
packages/python-sdk/ — Python SDK (PyPI: lastsearch)
src/               — Vite + React frontend (landing, developers, playground pages)
```

> The verification engine (Fastify REST API) has been moved to a separate private repository: [lastsearch-hq/lastsearch-engine](https://github.com/lastsearch-hq/lastsearch-engine). The public repo's frontend proxies `/api/*` requests to the hosted engine.

## Two-Repo Architecture

This project spans **two repositories**. Always consider both when making changes:

| Repo | Visibility | What lives here | Deploys to |
|------|-----------|-----------------|------------|
| **lastsearch** (this repo) | Public (Apache 2.0) | MCP server, Python SDK, frontend, shared types, examples, docs | Vercel (frontend + MCP function) |
| **lastsearch-engine** (`/tmp/lastsearch-engine`) | Private (BSL 1.1) | Verification engine, API routes, NLI, confidence scoring, Redis cache, Supabase | Vercel (separate project) |

**When making changes:**
- **API parameter changes** → update engine routes + MCP tool schemas + Python SDK methods + shared types
- **New features** → engine implementation + public repo surfaces (MCP, SDK, docs, frontend)
- **Auth/key changes** → both repos (engine auth middleware + MCP/SDK client code)
- **Version bumps** → public repo only (MCP package.json + index.ts, Python SDK pyproject.toml + __init__.py)

**All API access requires a LastSearch key (`ls_xxx`).** There is no BYOK (bring your own keys) mode. The engine handles all search and LLM calls server-side.

## Key commands

```bash
pnpm dev:web          # Frontend only (Vite)
pnpm build            # Full build (Vercel)
pnpm test             # Run tests (vitest)
npx pnpm --filter lastsearch build  # Build MCP only
```

> The API server is hosted separately (lastsearch-engine repo). There is no `dev:api` command in this repo.

## Architecture decisions

- **Engine deployment:** The verification engine (API server) lives in a separate private repo ([lastsearch-hq/lastsearch-engine](https://github.com/lastsearch-hq/lastsearch-engine)) and is deployed as a hosted service. This public repo's frontend proxies `/api/*` requests to the engine. All API access requires a `ls_xxx` key.
- **LLM:** Google Gemini 2.5 Flash via OpenRouter (`packages/shared/src/constants.ts`)
- **Search:** Multi-provider search — primary search API + secondary provider for source diversity (parallel execution, deduplicated results).
- **Verification pipeline:** Hybrid BM25 + dense embeddings + NLI semantic entailment → cross-source consensus → NLI contradiction detection. Falls back to BM25-only when premium keys not set.
- **Embedding retrieval:** OpenAI `text-embedding-3-small` via OpenRouter for semantic candidate retrieval. BM25 + embedding rankings fused via Reciprocal Rank Fusion (RRF). Catches paraphrased claims BM25 misses. Premium tier only, graceful BM25 fallback.
- **NLI reranking:** Top-3 RRF-fused candidates per claim reranked by DeBERTa NLI entailment scores. Picks best supporting evidence semantically, not just by keyword match.
- **Atomic claim decomposition:** Compound claims auto-split into individual verifiable facts before verification. Splitters: `and`, `;`, `while`/`whereas`/`but`.
- **Multi-pass consistency:** In thorough mode, claims cross-checked across two independent extraction passes. Confirmed claims boosted (+0.08), unconfirmed penalized (-0.05). SelfCheckGPT-inspired.
- **Confidence scores:** 8-factor evidence-based algorithm — NOT LLM self-assessed. Auto-calibrated from user feedback via isotonic regression (70% calibrated + 30% original blending). Factors: verification rate (22%), domain authority (18%), source count (15%), consensus (12%), domain diversity (10%), claim grounding (10%), source recency (8%), citation depth (5%). Contradiction penalty applied.
- **Domain authority:** 10,000+ domains in Supabase (260 curated + Majestic Million), 5-tier scoring with Bayesian dynamic blending from real query verification data. Cold-start safe via prior weight smoothing.
- **Thorough mode:** `depth: "thorough"` runs iterative confidence-gated loop (FIRE-inspired, up to 3 passes) with per-claim evidence retrieval (SAFE-inspired) and counter-query adversarial verification (SANCTUARY-inspired). Early termination via query similarity detection. Runs multi-pass consistency checking. Available across API, MCP, and Python SDK.
- **Per-claim evidence retrieval:** After initial extraction, weak claims (unverified or low score) get targeted LLM-generated search queries. Each claim gets its own evidence from all providers (Tavily + Brave + Exa). Boosted claims get new sources added. Premium tier only.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LastSearch-HQ/lastsearch](https://github.com/LastSearch-HQ/lastsearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
