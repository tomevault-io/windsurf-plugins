---
trigger: always_on
description: Cryptocurrency futures short-signal radar. Scans 150+ Binance USD-M futures every 5 minutes,
---

# DAO VANG (PeakPulse AI) — Project Context

## What this project does

Cryptocurrency futures short-signal radar. Scans 150+ Binance USD-M futures every 5 minutes,
scores distribution risk (0–100), delivers alerts via Telegram + React dashboard,
and optionally executes trades via an isolated Execution Engine (Testnet paper-trading first).

---

## Architecture — pipeline flow

```
Binance REST
  → Collectors (klines/OI/funding/taker/ratios)
  → DuckDB + Parquet storage
  → Feature builders (point-in-time, zero lookahead)
  → Frozen model bundle (LightGBM + isotonic calibration)
  → DistributionScorer (composite 0-100)
  → Quality gate (>= 70% calibrated probability)
  → Telegram alerts + REST API + React UI
  → [Optional] ExecutionEngine (Auto-trading, Testnet-first)
```

### Key module map

| Path | Responsibility |
|---|---|
| `src/dao_vang/config/settings.py` | `AppSettings` via Pydantic v2 + env vars |
| `src/dao_vang/data/collectors/` | Per-metric Binance collectors |
| `src/dao_vang/data/storage/duckdb.py` | DuckDB query layer |
| `src/dao_vang/features/builder.py` | Point-in-time feature assembly |
| `src/dao_vang/features/builders/` | Per-signal builders (price, OI, funding) |
| `src/dao_vang/scoring/distribution_scorer.py` | Composite 0-100 score (rule components) |
| `src/dao_vang/scoring/frozen_inference.py` | Frozen model serving, fail-closed |
| `src/dao_vang/scanner/daemon.py` | `ScannerDaemon` — 24/7 main loop |
| `src/dao_vang/scanner/candidate_filter_v2.py` | Pre-filter before model scoring |
| `src/dao_vang/scanner/pump_filter.py` | Pump pattern pre-filter |
| `src/dao_vang/scanner/anomalies.py` | Independent anomaly radar (0-100) |
| `src/dao_vang/scanner/outcomes.py` | Empirical precision/PnL tracking |
| `src/dao_vang/alerts/telegram.py` | Telegram delivery + dedup |
| `src/dao_vang/execution/` | Isolated Auto-Trading Execution Engine |
| `src/dao_vang/web/api_server.py` | ThreadingHTTPServer REST API |
| `src/dao_vang/validation/` | Walk-forward splitter, leakage auditor |
| `src/dao_vang/experiments/` | Training runner, forward test, self-learning |
| `frontend/src/` | React 19 + TypeScript + Vite dashboard |

---

## Environments

| Env | DB | Ports | Mode |
|---|---|---|---|
| Dev | `data/dev.duckdb` | API: 8000, UI: 5173 | `research` — no Telegram sends |
| Live | `data_live/live.duckdb` | API: 8001 | `live` — real alerts |

Do not mix `data/` and `data_live/` paths. Never write to `data_live/` during research/testing.

---

## Build, run, and test commands

```bash
# Install / sync dependencies (Python 3.12, uv)
uv sync --all-extras --dev

# Run dev (Windows batch — starts API on :8000 + Vite on :5173)
.\run_dev.bat

# Run API backend only
python -m dao_vang.web.run --port 8000

# Run scanner only
python -m dao_vang.cli.main scan

# Backend tests (full suite)
uv run pytest

# Specific test path
uv run pytest tests/unit/scoring/

# Linting
uv run ruff check .

# Type checking (only these files are gated in CI)
uv run pyright src/dao_vang/config/settings.py src/dao_vang/scoring/frozen_inference.py src/dao_vang/updater/manager.py src/dao_vang/web/version_history.py

# Frontend
cd frontend && npm ci && npm run build
```

---

## Non-negotiable correctness rules

### 1. Zero lookahead bias — absolute
- Feature builders consume only data at or before candle close timestamp `t`.
- DuckDB time-series joins use `as-of` matching (strictly `<=`). Never use `>=` on future rows.
- Walk-forward splits use embargo gaps. Test fold labels must **never** reach threshold selection
  (enforced in `tests/qa/test_release_gates.py::test_threshold_helper_never_receives_test_labels`).

### 2. Frozen model bundles — immutable in production
- `scanner.frozen_model_id` in config points to a locked artifact directory.
- `frozen_inference.py` verifies SHA-256 checksums before scoring. **Never bypass checksum verification.**
- Raw model probabilities must pass calibration gate before alerting (`calibrator_unvalidated_identity`
  reason code = not alertable).
- Self-learning (`self_learning.enabled`) creates challenger models only; it never overwrites
  `scanner.frozen_model_id`. That key is changed only by an explicit human decision.

### 3. Scoring fail-closed
- Stale snapshots (`max_feature_age_minutes = 10`) → `alertable = False`.
- Invalid quality status → `alertable = False`.
- `FrozenInferenceError` propagation must not be swallowed silently.

### 4. Telegram rate limits — always enforce
- `global_daily_alert_limit = 15`, `coin_daily_alert_limit = 2`.
- `cooldown_minutes = 120` per coin before re-alert.
- `operating_mode: research` must never send real Telegram messages.
- `shadow_telegram_enabled` in research mode sends to a labelled shadow channel only.
- Any code path that calls `TelegramNotifier` must check `_mode_allows_tier()` first.

### 5. Candidate comparison challenger — audit only
- The challenger in `candidate_comparison` logs outcomes only. It must never trigger Telegram sends.

### 6. Auto-trading safety — isolated execution
- `ScannerDaemon` must **never** execute trades directly. It only produces `alertable = True` signals.
- `ExecutionEngine` runs as a completely decoupled component, reading signals from the database or via webhook.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrcanlaco/PeakPulse-AI](https://github.com/mrcanlaco/PeakPulse-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
