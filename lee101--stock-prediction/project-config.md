---
trigger: always_on
description: - Every new model / variant / training knob is measured against **median
---

# Agents Guide

## PRODUCTION GROUND RULES (read this before touching anything live)

### Target: ≥27% monthly PnL on 100+ day unseen data
- Every new model / variant / training knob is measured against **median
  monthly return ≥ 0.27** on the worst slippage cell of
  `scripts/eval_100d.py` run with `decision_lag=2` binary fills.
- Under-target variants are not production. Don't merge into the 32-model
  ensemble, don't update `alpacaprod.md`, don't flip `ALLOW_ALPACA_LIVE_TRADING=1`.
- Use the fail-fast gates (`--fail-fast-max-dd 0.20`, negative-month bail,
  skip-video-on-dud) so doomed experiments cost ~3s instead of ~3min.

### Single-writer rule: one live Alpaca process, server-enforced
- Every process that reaches Alpaca's write API **must** import
  `alpaca_wrapper`. At import time, `src/alpaca_singleton.py::enforce_live_singleton`
  acquires an fcntl writer lock on `<state>/account_locks/alpaca_live_writer.lock`.
  A second live import **exits 42** with a loud stderr banner naming the
  holder PID/host/start-time.
- Paper mode (`ALP_PAPER=1`) skips the lock entirely — run as many paper
  clients as you like. Live (`ALP_PAPER=0`) enforces singleton.
- Override escape hatch: `ALPACA_SINGLETON_OVERRIDE=1`. This is a
  break-glass lever that logs `OVERRIDE ACTIVE` to stderr on every order
  and should NEVER be set in systemd units — only by a human at a terminal
  during a conscious recovery.
- Do not add a second path that reaches Alpaca. If you need a new entry
  point (REST server, webhook, CLI) make it import `alpaca_wrapper` so it
  inherits the gate.

### Death-spiral guard: no selling below last buy
- `alpaca_wrapper.alpaca_order_stock` calls
  `src/alpaca_singleton.py::guard_sell_against_death_spiral` before every
  order. Tolerance is **time-aware**:
  - **Intraday** (buy ≤8h old): any sell more than **50 bps below** the
    last recorded buy raises `RuntimeError` — stops the "keep lowering
    the ask until we fill" intra-session death loop.
  - **Overnight / hold-through** (buy >8h old): tolerance widens to
    **500 bps** so a normal overnight gap-down on a rotated position
    does not crash the daemon in a restart loop. A >5% gap still
    refuses — that's the cliff where "something is genuinely wrong"
    (halt, delisting, bad fill).
  - Explicit `tolerance_bps=` on the call overrides regime selection
    (back-compat path for callers that already know their regime).
- Buy prices are tracked per-symbol on disk under
  `<state>/alpaca_singleton/alpaca_live_writer_buys.json` so the guard
  survives restarts. The record window is 3 days; older buys are pruned.
- Override escape hatch: `ALPACA_DEATH_SPIRAL_OVERRIDE=1`. Same rules —
  loudly logged, never in systemd units, only by a human.

### Before changing any of the above
- `tests/test_alpaca_singleton.py` must stay green. If you need to change
  the guard semantics, update the tests in the same commit.
- Update this section + the matching block in `CLAUDE.md` in lockstep.

## Machines
- `local`: development machine
- `leaf-gpu`: see `~/.secretbashrc` for connection details

## Secrets
- Secrets live in `~/.secretbashrc` on each machine and are gitignored.
- Source that file before running trading bots.

## Model Checkpoints
- Model weights (`*.pt`, `*.pth`, `*.safetensors`) are gitignored.
- Sync them via R2/S3 instead of git.

```bash
# Upload checkpoint to R2
rclone copy pufferlib_market/prod_ensemble/ r2:model/stock/prod_ensemble/ --progress

# Download checkpoint from R2
rclone copy r2:model/stock/prod_ensemble/ pufferlib_market/prod_ensemble/ --progress

# Sync specific model
rclone copy pufferlib_market/checkpoints/some_run/best.pt r2:model/stock/checkpoints/some_run/

# Full sync between machines
rclone sync r2:model/stock/ ./model_sync/ --progress
```

- Configure `rclone` for Cloudflare R2 with `rclone config`.

## Workflow
- Work on `main`; do not create feature branches for routine changes here.
- Prefer `uv pip`; do not use bare `pip`.
- Prefer activating an existing virtualenv and then using normal `python` / `pytest` over `uv run`.
- This is a monorepo for trading experiments; it is normal to work across multiple subprojects.
- We keep multiple environments around, including `.venv`, `.venv312`, and `.venv313`. Keep them working when practical.
- Do not cut work short because installation or test setup is inconvenient. Finish the task end to end.
- Long-running training or benchmark jobs are expected; do not add arbitrary short timeouts that break real workloads.
- Write and run tests aggressively while developing.

## Git Workflow
- Large binary artifacts go to R2, not git.
- After a force-push, remote machines should realign with `git fetch origin && git reset --hard origin/main`.

## Environment Setup
```bash
source ~/.secretbashrc
source .venv313/bin/activate  # or .venv / .venv312 when appropriate
uv pip install -e .
```

## Engineering Standards
- Keep changes correct, minimal, and robust on edge cases.
- Prefer simple functions and straightforward control flow over unnecessary inheritance or abstraction.
- Fix root causes instead of masking symptoms with blanket workarounds.
- When normalizing inputs, preserve API contracts and compatibility with built-ins and major library types.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lee101/stock-prediction](https://github.com/lee101/stock-prediction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
