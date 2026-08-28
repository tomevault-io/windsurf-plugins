---
trigger: always_on
description: > **Language: English (en)** — This document is written in en only.
---

> **Language: English (en)** — This document is written in en only.
# AGENTS.md — Guide for AI Agents Working in This Repository

This file tells AI agents (and humans) everything needed to read, validate, and update
this repository correctly. Read it fully before making changes.

## ⚠️ Project Status

**This repository is a work in progress.** Data may be outdated, incomplete, or wrong;
some billing modes are hard to verify. Treat every entry as "as-of" data:
- check `verified_at` / `updated_at` and the `notes` (source) before trusting a number;
- `null` means unknown/not offered — never invent a value, never use 0 for "unknown";
- subscription-included models have `per_mtok: null` + a note (never 0);
- deprecated/retired models carry `"status"` and remain as historical entries.

Contributions are welcome via issues and PRs (see `CONTRIBUTING.md`); human changes go
through PRs checked by `.github/workflows/pr-check.yml`. Bot syncs merge straight into `main`.

**How this project is built**: maintained with
[DeepSeek Harness](https://github.com/deepseek-ai/DeepSeek-Harness) using the
**deepseek-v4-flash-0731** model.

## What This Repository Is

`ai-model-pricing` is an open database of **AI model pricing** covering every obtainable
channel: first-party vendor APIs (per-MTok, cache, batch), image/audio pricing, credit
systems, GPU-hour pricing, consumer subscriptions, and coding-tool plans.

- Machine-readable data: `data/feed/` (versioned JSON + JSON Schema)
- Human-readable pages: `data/view/` (Markdown, **generated** — never edit by hand)
- Auto-updated every 3 hours by GitHub Actions: `.github/workflows/daily-check.yml` (changes merge straight into `main`; no review needed for bot syncs)

## Repository Layout

```
data/feed/
  schema.json            # THE authoritative JSON Schema (26.0.1)
  index.json             # Entry point: providers/resellers lists, counts, timestamps
  providers/*.json       # One file per provider (provider_id.json)
  plans.json             # Subscription & coding-tool plans
data/meta/
  manifest.json          # Sync health: sources, last_ok/last_error
  changelog.json         # Every change (add/update/remove/verify), newest first
data/view/              # GENERATED (never edit): en/*.md + zh-CN/*.md
docs/                    # providers.md (landscape & status, generated), price-types.md,
                         # research-contract.md, verification.md
scripts/
  router.py              # Core check router: discovers checks/, runs in tier order
  toolbox.py             # Shared utilities (http, JSON, changelog, manifest, dedup)
  checks/                # Per-provider official-price checks (tierN_<provider>.py)
  daily_check.py         # Daily entry: router (official) -> models.dev -> OpenRouter
  sync_official.py       # Standalone official-source sync (official_sources.json registry)
  sync_openrouter.py     # OpenRouter catalog sync (aggregator prices)
  sync_modelsdev.py      # models.dev catalog sync
  validate.py            # Schema + consistency validation
  audit.py               # Repo-wide audit (version, counts, zero-price, docs bilingual)
  build_human.py         # Generate human pages (en + zh-CN)
  stats.py               # Exact data statistics for README
  bump_version.py        # Version bump (year.content.feature) + changelog entries
  merge_research.py      # Merge research-subagent JSON output
CONTRIBUTING.md          # contribution guide (en + zh-CN)
```

## Reading Data (for agents building tools)

1. Fetch `data/feed/index.json` first. Check `schema_version` (major bump = breaking).
2. Each `providers[]` / `resellers[]` entry has `file` (relative path), `model_count`, `updated_at`.
3. Model shape: `{id, name, category, status, modalities, context_window, max_output, billing_model, pricing, notes}`.
   `status` = **online | offline** only. Offline models keep the reason (retired/deprecated/superseded)
   in `notes` and stay as historical entries with a ❌ mark in the human pages.
   `billing_model` (required, array) = how the model is billed; one model can have several:
   `pay_per_token` (per-token API, incl. cache/batch), `pay_per_image`, `subscription_included`
   (included in a subscription/coding plan), `credits` (points-based), `free`, `unknown` (needs review).
   Use `python scripts/annotate_billing.py` to (re-)annotate; audit flags unknown/pay-per-token inconsistencies.
4. `pricing` fields (all USD per 1M tokens unless `currency` says otherwise):
   - `per_mtok.{input,output,cache_read,cache_write}` — per-token API prices
   - `batch.{input,output}` — 50% off batch APIs
   - `per_image[]` — tiers for image models
   - `promo.{list_price, ends_at}` — temporary discount; current `per_mtok` is the promo price,
     `list_price` holds the pre-promo value and `ends_at` the expiry (UTC ISO)
   Other billing fields (per_audio_second, per_character, per_request, credits, gpu, neuron_second,
   finetune, provisioned) were REMOVED from the schema on 2026-08-28 because nothing used them.
   **To add a billing mode back**: (a) add the field to `schema.json#/$defs/modelPricing.properties`,
   (b) add its value to `$defs.priceType.enum` and `$defs.billingModel.items.enum`, (c) populate real

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duanjiangDJ/ai-model-pricing](https://github.com/duanjiangDJ/ai-model-pricing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
