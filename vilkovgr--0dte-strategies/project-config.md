---
trigger: always_on
description: > Context file for AI coding agents (GitHub Copilot, OpenAI Codex, Claude, etc.)
---

# AGENTS.md — AI Agent Onboarding

> Context file for AI coding agents (GitHub Copilot, OpenAI Codex, Claude, etc.)
> working inside this repository.

## Mission

This is the **public replication package** for the paper *"0DTE Trading Rules: Tail Risk, Implementation, and Tactical Timing"* (Vilkov, 2026). The repo enables three tiers of engagement:

1. **Instant replication** — run analysis scripts against shipped derived data to reproduce every table and figure.
2. **Rebuild from source** — use Massive or ThetaData API adapters to reconstruct raw data from scratch.
3. **Explore and extend** — use AI-optimized context to understand, critique, or extend the analysis.

## Paper Summary (One Paragraph)

The paper studies realized payoffs of S&P 500 zero-days-to-expiration (0DTE) options and standard multi-leg structures from 09/2016 to 01/2026. A positive 0DTE variance risk premium exists but is small after realistic frictions. Strategy PNL distributions are wide, tail-heavy, and regime-dependent — dominated by directional and skewness realizations rather than stable mean carry. Yet disciplined 10:00 ET conditional rules under a strict out-of-sample protocol deliver economically meaningful net performance for selected strategies (put ratio spreads SR ≈ 1.26, iron butterflies SR ≈ 0.82) and diversified baskets (SR 1.01–1.27). The practical implication: 0DTE is better viewed as a tightly risk-budgeted tactical overlay than a standing carry strategy.

## Three-Tier Data Model

```
Tier 1 (shipped):    data/*.parquet, data/*.csv
                     ↓ analysis scripts read these directly
                     output/tables/*.tex, output/figures/*.pdf

Tier 2 (rebuild):    API key → code/ingest/{massive,thetadata}/
                     → raw snapshots → code/build_data.py
                     → data/*.parquet (same schema as Tier 1)

Tier 3 (explore):    docs/agent-context/* + AGENTS.md + CLAUDE.md
                     → LLM understands paper claims, methods, variables
```

## Repository File Map

```
code/
  config.py              Path configuration (replaces internal zEnvmt setup)
  run_replication.py     Single entry point: runs all analysis in order
  analysis/
    _paths.py            Shared path resolution helpers
    option_strats_uncond_analysis.py    → Tables 1a–1e (unconditional returns)
    compute_implementable_pnl.py       → Table 2 (execution costs, turnover)
    compute_tail_risk_diagnostics.py   → Table 3 (tail risk, capital at risk)
    compute_clustered_inference_mht.py → Table 4 (clustered SE, BHY correction)
    compute_structural_break_2022.py   → Table 5 (structural break test)
    compute_vix_regime_conditioning.py → Table 6 (VIX regime splits)
    compute_conditional_oos_protocol.py → Table 7 (OOS conditional results)
    compute_conditional_model_zoo.py    → Tables 8–9 (model comparison zoo)
    compute_conditional_oos_investment_ts.py → OOS investment time series
    build_conditional_target_choice_table.py → Target choice comparison
    derive_binary_decision_summary.py   → Binary decision framework
    moneyness_selection.py              → Representative moneyness selection
    plot_conditional_topk_basket_legs.py → Top-K basket composition
    figs_strats.py                      → All figures (payoffs, bars, time series)
  ingest/
    massive/             Massive API download adapter
    thetadata/           ThetaData API download adapter

data/                    Shipped derived panels (Git LFS)
  data_opt.parquet       Interpolated option panel (moneyness 0.98–1.02)
  data_structures.parquet Strategy panel (7 types × moneyness × dates)
  vix.parquet            VIX + intraday moments
  slopes.parquet         Volatility surface slopes (PIT)
  future_moments_SPX.parquet  Forward realized moments, PIT intraday (SPX)
  future_moments_VIX.parquet  Forward realized moments, PIT intraday (VIX)
  ALL_eod.csv            End-of-day reference prices

output/
  tables/                Generated LaTeX tables
  figures/               Generated PDF/PNG figures

tests/
  reference/tables/      Ground-truth LaTeX for parity checks
  fixtures/sample_paper.tex  Self-contained LaTeX fixture for CI
  test_replication.py    Byte-level parity test (tables)
  test_tools.py          Smoke tests: doctor, converter, doc structure

tools/
  doctor.py              Environment health checker (base→FAIL, model-zoo→WARN)
  latex2md.py            LaTeX → annotated-markdown converter (--meta overrides)
  sync_to_public.py      Dev-to-public sync helper

docs/
  agent-context/         Structured context for LLM agents
    reading-guide.md     Paper reading guide: claims, structure, methods
    variables.md         Variable definitions and feature families
    method.md            Methodology overview
    results-summary.md   Key results summary
    strategies.md        Strategy type descriptions
  manifests/             Exhibit-to-code mappings
    exhibits.csv         Table/figure → script mapping
    data-files.csv       Data file inventory
    code-map.md          Code structure narrative
    paper-claims.csv     Paper claims → evidence mapping
  paper/
    paper-annotated.md   Full paper with @section-type annotations
    paper-digest.md      Condensed paper digest
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vilkovgr/0dte-strategies](https://github.com/vilkovgr/0dte-strategies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
