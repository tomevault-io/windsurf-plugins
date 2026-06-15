---
trigger: always_on
description: Modeltell is a computational linguistics project that systematically measures the lexical and syntactic fingerprints of large language models. Unlike surface-level "AI detector" tools or meme lists about "delve", this project analyzes **grammatical constructions** — the sentence structures, rhetorical patterns, and compositional habits that define each model's linguistic identity.
---

# CLAUDE.md — Modeltell

## Vision

Modeltell is a computational linguistics project that systematically measures the lexical and syntactic fingerprints of large language models. Unlike surface-level "AI detector" tools or meme lists about "delve", this project analyzes **grammatical constructions** — the sentence structures, rhetorical patterns, and compositional habits that define each model's linguistic identity.

The key differentiator is the **syntactic pattern layer**: not just which words LLMs overuse, but which sentence structures they default to (tricolons, hedging openers, pseudo-inclusive "Whether you're X or Y" constructions, em-dash dramatic pivots, etc.).

This is an open source project by Third Shift Lab. The data, analysis, and CLI tool are MIT-licensed. A SaaS layer with MCP integration is planned for later (see "Future: SaaS Layer" below) but is NOT part of the current scope.

## Project Structure

```
modeltell/
├── CLAUDE.md                          ← You are here
├── README.md                          ← Public-facing docs
├── package.json
├── tsconfig.json
├── .env.example
├── .github/workflows/monthly-run.yml  ← Automated monthly re-runs
│
├── prompts/
│   └── catalog.json                   ← 30 standardized content prompts
│
├── patterns/
│   └── definitions.json               ← 15 syntactic patterns + lexical watchlists
│
├── src/
│   ├── runner/
│   │   ├── types.ts                   ← Shared type definitions
│   │   ├── models.ts                  ← Model registry (12 models, 3 tiers)
│   │   └── runner.ts                  ← Multi-provider API runner
│   │
│   ├── analysis/
│   │   ├── lexical.ts                 ← TF-IDF, word frequency, n-grams
│   │   └── syntactic.ts              ← Grammar pattern detection, structural metrics
│   │
│   ├── publish/
│   │   └── publish.ts                 ← Transforms analysis into community-friendly JSON
│   │
│   └── cli/
│       └── check.ts                   ← CLI linter: `npx modeltell check "text"`
│
├── data/                              ← Raw outputs per run (gitignored; only published/ is committed)
│   └── {run-id}/
│       ├── manifest.json
│       ├── {model-id}.json            ← Raw generations
│       ├── _lexical_analysis.json
│       └── _syntactic_analysis.json
│
├── published/                         ← Community-friendly data (auto-generated)
│   ├── index.json
│   ├── models/{model-id}.json
│   ├── watchlist/words.json
│   ├── watchlist/constructions.json
│   ├── comparisons/tier-summary.json
│   └── comparisons/similarity-matrix.json
│
└── frontend/                          ← Scrollytelling visualization (TODO)
```

## Stack

- **Pipeline**: TypeScript, tsx, Node.js 22+
- **APIs**: Anthropic, OpenAI, Google Gemini, Together AI (Llama/DeepSeek/Qwen), Mistral
- **Frontend**: React, Vite, Framer Motion, D3.js or Recharts
- **Deployment**: Vercel (frontend), GitHub Actions (pipeline), GitHub Pages (data API)
- **No database** — JSON files; the curated `published/` dataset is committed (raw `data/` is gitignored to keep the repo small)

## Models Tracked

| Tier | Models | Provider |
|------|--------|----------|
| Frontier | Claude Opus 4, Claude Sonnet 4, GPT-4o, GPT-4.1, Gemini 2.5 Pro | Anthropic, OpenAI, Google |
| Mid | Claude 3.5 Haiku, GPT-4.1 Mini, Gemini 2.5 Flash | Anthropic, OpenAI, Google |
| Open Source | Llama 3.3 70B, DeepSeek V3, Mistral Large, Qwen 2.5 72B | Together, Mistral |

Models are defined in `src/runner/models.ts`. Adding a new model = adding one entry to the registry.

## Pipeline Flow

```
1. GENERATE   prompts/catalog.json → runner.ts → data/{run-id}/{model}.json
2. ANALYZE    data/{run-id}/ → lexical.ts + syntactic.ts → _*_analysis.json
3. PUBLISH    data/{run-id}/ → publish.ts → published/
4. VISUALIZE  published/ → frontend → deployed website
```

Each step is independent and idempotent. You can re-run analysis without re-generating. You can re-publish without re-analyzing.

## Commands

```bash
npm install

# Generation (needs API keys in .env)
npm run run:all                          # All 12 models
npm run run:anthropic                    # Just Anthropic models
ONLY_MODELS=gpt-4o npm run run:all      # Single model

# Analysis
npm run analyze:lexical -- data/{run-id}
npm run analyze:syntactic -- data/{run-id}

# Publish community data
npm run publish:data -- data/{run-id}

# CLI linter
npm run check -- "Your text here"
npm run check -- --file input.md --json
```

## What's Done

- [x] Prompt catalog (30 use cases, 5 categories)
- [x] Pattern definitions (15 syntactic patterns + lexical watchlists)
- [x] Multi-model runner with 5 provider adapters
- [x] Lexical analyzer (TF-IDF, frequencies, n-grams)
- [x] Syntactic analyzer (pattern matching, openers/closers, structural metrics, radar profiles)
- [x] Data publisher (community JSON format with schema versioning)
- [x] CLI linter with scoring, grades, and actionable tips

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thirdshiftlab/modeltell](https://github.com/thirdshiftlab/modeltell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
