---
trigger: always_on
description: - **No em dashes. Ever.** Never write "—". No AI-sounding filler. Write like a human, in the user's casual style. Short, direct, not machine-like.
---

# AGENTS.md

## Rules (always)

- **No em dashes. Ever.** Never write "—". No AI-sounding filler. Write like a human, in the user's casual style. Short, direct, not machine-like.
- **Assume nothing. Guess nothing. Make no mistakes.** Always give 100% effort. If unsure, verify or ask. Never paper over a gap with a plausible-sounding guess.

## Repo status

- Scaffolded for plan §5 Days 1–14 + Days 15–35 (entry + training stack): Python package `src/nirnay/**` (incl. concepts/deepsup/rlcd/coarse2fine + data/lora/losses/train/eval), gate scripts `scripts/check_*.py` (G1–G19 plus stabilization checks), `GATES.md` (**19/19 met** with automatic evidence), `pyproject.toml` + `uv.lock`. Train/serve stabilization now loads trainable checkpoints into `NirnayAgent`, applies the same NoPE fraction in training/serving, conditions the encoder on byte patches, uses c2f for 77-way Banking77, and persists resumable Phase A checkpoints. Git `main` → private **https://github.com/eulogik/nirnay** (root commit 1047a72; push with `git push origin main`). No lint/typecheck/CI yet; do not invent them.
- **Phase A v1 (2026-09-24) failed eval at exactly 1/77; diagnosis + fixes landed 2026-09-25** (see MEMORY.md "Phase A v1 failure" + "Fixes landed"): dense-scorer top-20 stage-1, fused pointer, eval no-forcing, seeded shuffle, train-only mix, identity-at-init byte/concepts. Second-round fixes (2026-09-25 evening, same file): byte LayerNorm-before-projection, **concepts delta LayerNorm removed (post-zero-init LN amplification caused eval cliff at steps 6–9)**, **three-group optimizer `--pretrained-lr 1e-5 --concepts-lr 1e-4` (pretrained diffusion + VQ spike 22 at step150 caused second cliff 100–150)**; combo 250-verify: dense 0.5195 / top20 0.9156 / delivered 0.5195 (baselines 0.1429/0.5844/0.1461). Our own Banking77 zero-shot baselines: dense_acc 0.1429, top20 recall 0.5844 (plan's cited "Laya 0.425" not reproducible under our template; report our own numbers). Heavy compute (training/MPS) requires asking the user first.
- Banking77 CSVs (CC-BY-4.0, PolyAI task-specific-datasets) live in `data/banking77/{train,test}.csv` (tracked; do not delete).
- `MEMORY.md` holds session state, verified facts, and open risks. Read it before starting work.

## Source of truth

- `NIRNAY-Breakthrough-Plan.md` defines the product: frozen v1 architecture (§1), training recipe (§2), pre-registered benchmarks (§3), 90-day build plan (§5), kill gates (§6).
- Paper/data cutoff: sources through **Sept 19, 2026** (Cheap Verifiers arXiv:2609.01345 v2 verified Sept 23); plan verified/amended **Sept 23, 2026** (amendment log at file end).
- If the plan conflicts with a freshly verified external source (arXiv, HuggingFace, JevBench): re-verify, fix the plan, append to its amendment log. Do not silently diverge.

## Non-negotiable (plan §0, §6)

- **License:** Apache-2.0. Fork only Apache-2.0 sources (Laya, pico-type).
- **Jev is closed:** compare only to published third-party numbers; never distill from Jev API outputs; never publish a Jev column as our own measurement.
- **Metrics:** always report raw **and** fitted ECE; keep zero-shot and fine-tuned results in separate columns.
- **Delivered error (2026-09-23):** never count a metric computed through our own act/escalate/abstain head as delivered-error evidence; delivered-error rates need an external ground-truth sample audit. Flywheel labels from external outcomes only; never verifier-rejected-tail self-training (plan §3/§6/§7, arXiv:2609.01345).
- **v1 scope frozen (plan §1):** NoPE head-masking only; GLA+RoPE split is v1.1 (120M distill target). Params ~450M = 421M Laya fork (LoRA) + ~30M additions. Do not add components absent from §1 without updating the plan first.
- **Benchmarks (§3) are pre-registered:** no cherry-picking; publish pass/fail even on miss.

## Verification discipline

- Citations and benchmark numbers in the plan have been wrong and fixed; see the Sept 22 amendment log. Fetch/spot-check arXiv IDs and leaderboard rows before relying on them.
- Gate runner: unlazy `gate-check.mjs`. Approvals live under `~/.unlazy/approved` (outside repo). EXPECT must be a literal output prefix; comparison operators do not match.
- Model weights: Laya 421M in HF cache (`HF_HOME=/Volumes/KIOXIA 1TB/huggingface_cache`, blob sha256 `891102d372688fc2…`). Load offline with `HF_HUB_OFFLINE=1`. Download method of record: `aria2c -x 8 -s 8` (flaky network; parallel curl/hf hub stalled).

## Commands

Canonical order (from repo root, `uv` managed venv):

```sh
uv sync
uv run python -c "import nirnay; import nirnay.model; import nirnay.server; print('nirnay import ok')"   # G1
uv run python scripts/check_byte_path.py    # G3
uv run python scripts/check_nope_mask.py    # G4
uv run python scripts/check_temps_refit.py  # G6
uv run python scripts/check_forward.py      # G2 (needs model cache + HF_HUB_OFFLINE=1)
uv run python scripts/check_server_schema.py # G5
uv run python scripts/check_jevbench_public.py # G7 (NIRNAY_JEVBENCH_LIMIT=60 default)
uv run python scripts/check_concepts.py      # G8
uv run python scripts/check_deepsup.py       # G9
uv run python scripts/check_rlcd.py          # G10

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eulogik/nirnay](https://github.com/eulogik/nirnay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
