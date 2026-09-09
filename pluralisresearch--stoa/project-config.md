---
trigger: always_on
description: Guidance for anyone (human or AI agent) working in this repo. stoa runs **RL post-training with
---

# CLAUDE.md: stoa

Guidance for anyone (human or AI agent) working in this repo. stoa runs **RL post-training with
Apple-silicon Macs as the rollout fleet**: **MLX rollout workers** on the Macs generate rollouts, one
**CUDA slime/Megatron trainer** learns from them, and the two sides meet only at a **Cloudflare R2** bucket
(the PULSE weight-sync protocol), never addressing each other directly. Underneath, it's disaggregated,
off-policy GRPO.

## Layout

- `worker/`: the MLX rollout actor (`dRL_producer.py`), int8 decode, the continuous-batching scheduler
  (`cb_scheduler.py`), and the LFM2-only paged-KV Metal kernels (`native_kernels.py`, `paged_*.py`; the
  quickstart's Qwen path uses contiguous KV, so `DRL_KV_MODE=paged_prefix` is a no-op there).
- `common/`: R2 transport (`r2_*.py`), PULSE weight sync (`pulse_delta.py`, `pulse_sync.py`,
  `pulse_chain.py`), and the rollout codec.
- `trainer/`: slime custom functions: the rollout source (`r2_rollout.py`), the DPPO gate
  (`dppo_gate.py`), the Dr.GRPO reducer, replay buffer, staleness filter, eval.
- `patches/`: minimal source patches applied to THUDM/slime (`apply_slime_patches.py`).
- `envs/psqa_search/`: the PaperSearchQA multi-turn-search environment plus trainer launch scripts.
- `ops/`: node operations: one-command Mac join, retriever setup, fleet provisioning, supervisor.
- `bench/`, `scripts/`: benchmark harnesses and the data-fetch/build tooling.

Start at `README.md` (the runnable GSM8K quickstart) and
`runs/psqa-decoupled/REPRODUCE.md` (the reference write-up of the private 8B PSQA run, not a from-scratch recipe).

## Commits: Conventional Commits (required)

```
<type>(<scope>): <imperative subject>
```

- **types:** `feat`, `fix`, `chore`, `docs`, `test`, `refactor`, `perf`, `build`, `ci`.
- **scopes** (where useful): `worker`, `trainer`, `common`, `pulse`, `r2`, `slime`, `envs`, `bench`, `ops`, `scripts`.
- Subject: imperative, concise (≤ ~72 chars), no trailing period. Body explains the *why*.
- **Never** add AI/Claude attribution: no `Co-Authored-By`, no "Generated with", no emoji trailer.

## Environment & tooling

- Python **3.12**; **uv** is the package manager. `uv sync --extra worker` (or `retriever` / `data` /
  `dev`) installs a role's dependencies from `pyproject.toml`; `uv.lock` pins exact versions. The
  `retriever` extra (pyserini) requires Python 3.12, so install it into a 3.12 venv (`ops/setup_retriever.sh`
  creates one).
- The **worker** runs on Apple silicon (MLX); set it up with `ops/setup_mac_worker.sh` (creates the venv,
  applies the router patch, runs the int8 parity self-cert). The **trainer** runs inside the pinned
  slime/Megatron container, so its Python comes from the image, not uv.
- Most runtime environment variables use the `DRL_` prefix (historical; kept as the stable config
  contract); `R2_*`, `PSQA_*`, `HF_*`, and `LFM2_*` are also load-bearing.

## Lint & format

Config lives in `pyproject.toml`; run `pre-commit run --all-files` before committing. `ruff` (rules
E/F/B/UP), `black` and `isort` both at line-length 119 (isort `profile=black`). `E402` is ignored because
modules use a `sys.path` bootstrap before imports; keep the explicit `# noqa: E402` on those lines.

## Testing

- `pytest` at the repo defaults must stay **green on any machine**. Tests that need hardware or heavy deps
  guard themselves so they *skip* (not error) when the dep is absent: `pytest.importorskip("mlx.core")`
  for Metal-kernel tests, `importorskip("torch")` for trainer tests.
- The default `pytest` lane is offline and CPU-only. The heavier lanes are opt-in flags a bare run never
  triggers: `--run-realr2` (live Cloudflare R2, needs creds), `--run-metal` (Apple GPU), `--run-slow`
  (long batteries), `--run-realtok` (downloads the HF tokenizer). Never add a test that mutates live
  infrastructure without gating it behind the `realr2` marker (and its `--run-realr2` flag).
- Write **targeted, discriminating** tests: one behavior per test, named for what it proves, asserting the
  outcome that distinguishes the bug from the fix. A test that passes under both the bug and the fix is not
  a test.

## Load-bearing contracts (do not break these)

These are the invariants the system's correctness depends on. Read the module docstrings before touching
the code.

- **MLX lazy-evaluation ordering (`worker/paged_pool.py`, `native_kernels.py`).** The paged-KV pool is
  written in place, so ordering is enforced by hand: a returned **ordering token** is the read-after-write
  edge, **read stamps** guard write-after-read, and freed blocks are parked until a **drain** (`mx.eval`)
  confirms outstanding reads finished. Never wrap the kernels in `mx.compile`, never drop or reuse an
  ordering token, and derive stamps as copies (`+ 0`) so they do not pin parent buffers.
- **One weight version per cohort (`worker/dRL_producer.py`).** The worker pulls and swaps weights only
  *between* generation cohorts, then latches one immutable `(model, tok, version)` snapshot and generates
  the whole batch against it. A rollout group never spans two weight versions.
- **PULSE integrity (`common/pulse_*.py`).** Weights sync as periodic full **anchors** plus

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PluralisResearch/stoa](https://github.com/PluralisResearch/stoa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
