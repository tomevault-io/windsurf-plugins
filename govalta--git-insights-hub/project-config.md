---
trigger: always_on
description: Organization-wide GitHub analytics + a multi-detector security scanning pipeline. This file
---

# Git Insights Hub — repository guide

Organization-wide GitHub analytics + a multi-detector security scanning pipeline. This file
orients contributors (and Claude Code sessions) working in this repo. User-facing docs:
[README.md](README.md), [SCANS.md](SCANS.md), [docs/](docs/).

## Stack & conventions
- **Node 22, ESM** (`"type": "module"`). Express API + Vue 3 (CDN) dashboards in `public/`.
- **PostgreSQL 14+**; schema is created/updated by idempotent migrations in `db/migrate.js`
  (`npm run migrate`). Migrations are append-only — add a new numbered entry, never edit an
  applied one.
- **LLM provider** is pluggable (`lib/llm-provider.js`, `LLM_PROVIDER`): **Anthropic direct**
  (`anthropic`, standard commercial terms - for **unclassified** workloads) or **Claude on Google
  Vertex AI** (`vertex`, Enterprise Agent Platform, stays in the GCP enterprise tenancy under the
  enterprise agreement - for **Protected A/B** workloads). This classification mapping is the
  Government of Alberta's under its current STRA; it is deployer configuration, not a code property
  (see README "Choosing an LLM provider"). The two **opt-in** Agent-SDK paths
  (`AGENT_VERIFIER=1`, `bcm-scan --max`/`BCM_DEEPDIVE_AGENT=1`) use `@anthropic-ai/claude-agent-sdk`,
  which talks to Anthropic **direct** regardless of `LLM_PROVIDER`; both default off, so a Vertex
  Protected deployment stays on Vertex unless they are explicitly enabled.
  Tiers: `fast` = Sonnet (`claude-sonnet-5`), `deep` = Opus (`claude-opus-4-8`). Pick a tier
  with `getProviderForTier("fast"|"deep")`. **`temperature` is DEPRECATED on the flagship model**
  (Opus 4.8 on Vertex returns `400 "temperature is deprecated for this model"`), so it is NOT a
  usable determinism lever — the provider sends it only when explicitly opted in and drops it on
  the deprecation error. Run-to-run consistency of LLM steps must therefore come from *constraining
  the task* (compute numbers in code, tighten schemas), not from settings.
- **Decision-grade numbers are computed in code, never by the LLM.** The rebuild estimate lives in
  `lib/reporting/estimate.js` (`computeEstimate`): deterministic unit counts × per-unit rates parsed
  from `profiles/<org>.estimation.md` + a signal-derived complexity mix. Identical inputs → identical
  estimate (was ~34% drift when the model did the arithmetic in-prompt). `chief-architect.js` calls
  it and discards the model's `estimate`; the model only writes narrative. Tests: `tests/estimate.test.js`.
- **Reproducibility invariants (do not regress).** Because temperature
  can't pin the model, run-to-run consistency is engineered structurally: (1) every filesystem walk
  that feeds an LLM prompt or a persisted array is **sorted** (`file-inventory.js`, `import-graph.js`,
  `robust-scan.js` walks) and every `SELECT` feeding a prompt/dedup has an `ORDER BY`; (2) no LLM
  owns a **decision-grade number** — severity is derived in `consensus.js` from source detectors +
  a boolean verifier gate (not the verifier's tier enum), scores/estimate/disposition are computed
  in code, BCM primitive counts are **deduped** before counting, and all business-case option costs
  are code-locked; (3) LLM calls are **truncation-safe** — providers signal `{_truncated, partial}`
  and callers split/salvage instead of silently dropping (`code-scanner.js` splits batches); (4) the
  capability taxonomy is **order-invariant** (lexicographically-smallest canonical wins in
  `store.js`). Model-drawn Mermaid diagrams remain narrative (labeled as illustrative). Every LLM
  call retries all transient failures (5xx / network / rate-limit / empty-response) with backoff
  (`LLM_MAX_RETRIES`, default 5). For the residual set-membership drift in the two enumerated LLM
  stages, **multi-sample consensus** is opt-in via `CONSENSUS_SAMPLES` (N, default 1) + `CONSENSUS_MIN`
  (K, default majority): the AI generator (`code-scanner.js`) and BCM deep-dive (`deep-dive.js`) each
  sample N times and keep items recurring in >= K (pure helper `lib/util/consensus.js`).
- **Nothing org-specific is hardcoded** — it lives in an org profile (`profiles/*.json`,
  selected by `ORG_PROFILE`; `default.json` is the empty generic base, `example.json` is a complete
  generic worked example used by the test suite). Profile values are SQL-escaped via `lib/profile-sql.js`.
  Org-specific data that used to be hardcoded in shared code now lives in profile keys: repo→domain
  buckets (`dashboard_domain_patterns`, consumed by `dashboardDomainCase` in the dashboard SQL + the
  client drill-down via the capability-map API), content→domain keywords (`content_domain_keywords`,
  `capability-scanner.js`), external-system canonicalization (`external_system_aliases`,
  `architecture-map.js`), and repo family prefixes (`repo_family_prefixes`). All ship empty in
  `default.json`; an org fills in its own. Tests run under `ORG_PROFILE=example` (`tests/test-profile`),
  so the suite never depends on any real org's profile.
- **Single generic version / deployment & anonymity.** The tracked tree is 100% generic — no real
  system/repo names anywhere in committed code or config, so ONE codebase serves every org and any

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GovAlta/GIT-INSIGHTS-HUB](https://github.com/GovAlta/GIT-INSIGHTS-HUB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
