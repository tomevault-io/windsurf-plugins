---
trigger: always_on
description: **OmniOracle** — Motore di scoperta automatica di verità statistiche non banali da dati pubblici eterogenei.
---

# CLAUDE.md — OmniOracle

## Progetto
**OmniOracle** — Motore di scoperta automatica di verità statistiche non banali da dati pubblici eterogenei.

## Obiettivo
Costruire un engine che ingerisce decine di migliaia di serie temporali pubbliche cross-domain (economia, clima, salute, trasporti, brevetti, social), scopre automaticamente relazioni causali non banali tramite mutual information + causal discovery, e le presenta con rigore statistico completo (p-value, confidence interval, validazione out-of-sample).

## Stack Tecnico

| Componente | Tecnologia | Versione | Motivo |
|-----------|-----------|---------|--------|
| Linguaggio | Python | 3.12+ | Ecosistema data science maturo |
| Data processing | Pandas / Polars | latest | Gestione serie temporali |
| Statistical screening | Scikit-learn / Scipy | latest | MI, statistical tests |
| Causal discovery | DoWhy / CausalNex | latest | PC algorithm, DAG discovery |
| Information theory | NPEET / sklearn | latest | Mutual information KNN estimator |
| Time series | Statsmodels | latest | Granger causality, VAR models |
| Data ingestion | HTTPX / aiohttp | latest | Async API calls |
| Storage | DuckDB | latest | Analytics su dati colonnari, zero infra |
| Plausibility | Claude API | latest | LLM plausibility scoring |
| Viz | Plotly | latest | Grafici interattivi per output |
| Task orchestration | Luigi / Prefect | latest | Pipeline DAG |

> IMPORTANT: ogni dipendenza DEVE avere entry verificata in `verified-deps.toml`

## Architettura

```
                    FONTI PUBBLICHE
                    (API gov, open data, sensori)
                         │
                    ┌────▼────┐
                    │ INGEST  │  Async fetchers + normalizzazione
                    │ LAYER   │  → X_i(t, geo) serie temporali
                    └────┬────┘
                         │
                    ┌────▼────┐
                    │ STORAGE │  DuckDB: tabella universale
                    │         │  feature × tempo × geo
                    └────┬────┘
                         │
                    ┌────▼────────┐
                    │ DISCOVERY   │
                    │ ENGINE      │
                    │             │
                    │ L1: MI screening (scarta 99%)
                    │ L2: Lagged MI directional test
                    │    (rileva direzione + lag ottimale)
                    │ [Future: DAG discovery, Transfer Entropy]
                    └────┬────────┘
                         │
                    ┌────▼────────┐
                    │ VALIDATION  │
                    │ FILTER      │
                    │             │
                    │ FDR (Benjamini-Hochberg)
                    │ Out-of-sample temporal
                    │ LLM plausibility scoring
                    └────┬────────┘
                         │
                    ┌────▼────────┐
                    │ TRUTH       │
                    │ STORE       │  Verità validate con score,
                    │             │  grafo causale, fonti, CI
                    └────┬────────┘
                         │
                    ┌────▼────┐
                    │ OUTPUT  │  Dashboard, API, report
                    └─────────┘
```

## Moduli Core

```
omni-oracle/
├── src/
│   ├── __init__.py
│   ├── ingest/          # Fetcher per ogni fonte dati
│   │   ├── base.py      # Abstract fetcher
│   │   ├── fred.py      # Federal Reserve (800K+ serie)
│   │   ├── worldbank.py # World Bank (3K indicatori × 200 paesi)
│   │   ├── eurostat.py  # EU statistics
│   │   └── gdelt.py     # Global events
│   ├── storage/         # DuckDB schema + query layer
│   │   ├── schema.py
│   │   └── loader.py
│   ├── discovery/       # Il cuore statistico
│   │   ├── mi_screening.py      # Mutual Information screening
│   │   ├── lagged_mi.py         # Lagged MI directional test (pipeline principale)
│   │   └── granger.py           # Granger causality (solo in verify/, non nella pipeline)
│   ├── validation/      # Filtro anti-bullshit
│   │   ├── fdr.py               # Benjamini-Hochberg
│   │   ├── temporal_oos.py      # Out-of-sample validation
│   │   └── plausibility.py      # LLM plausibility scoring
│   ├── scoring/         # Ranking verità
│   │   └── ranker.py
│   └── output/          # Presentazione risultati
│       ├── truth_card.py
│       └── dashboard.py
├── tests/
├── verify/              # M4: verifica con tool alternativo
├── CLAUDE.md
├── verified-deps.toml
├── KNOWN_ISSUES.md
├── STATUS.md
└── Makefile
```

## MVP Scope

### IN (MVP — COMPLETATO)
- [x] Ingestione da 2 fonti (FRED + World Bank)
- [x] 49 serie FRED curate cross-domain
- [x] MI screening su tutte le coppie
- [x] Lagged MI directional test sulle coppie sopravvissute
- [x] FDR correction (Benjamini-Hochberg)
- [x] Out-of-sample temporal validation
- [x] Output: hypothesis cards con score, p-value, lag, fonti
- [x] Smoke test 5/5 PASS, golden snapshot L4 approvato

### F5 Roadmap (post-MVP)

#### Fase 0 (Mesi 0-3): Proprietary Trading + Credibilita' [IN PARALLELO]
- [x] Scaling a 500+ variabili (FRED + World Bank + Eurostat)
- [ ] Identificare top 10 relazioni con lag prevedibile per TRADING
- [ ] Backtest su dati storici (train/test split)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cesabici-bit/omni-oracle](https://github.com/cesabici-bit/omni-oracle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
