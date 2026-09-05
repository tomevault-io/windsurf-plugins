---
trigger: always_on
description: **Substitution project, not a redesign.** An existing P&ID intelligence agent already works
---

# PID-ML — Claude Code Context (Stage 4 Only)

## What this repo is

**Substitution project, not a redesign.** An existing P&ID intelligence agent already works
and uses cloud models (`claude-sonnet-4-6`, `claude-opus-4-8`, Google Cloud Vision) at several
stages. The goal is to replace each cloud call with the best **local** counterpart — a VLM,
LLM, or OCR model, fine-tuned per stage — then prove the local version matches the original.

**This repo is currently scoped to Stage 4 only.** Stage 4 (Symbol Detection) goes first
because it has full real ground truth and its winner becomes the **shared base VLM** every
other stage reuses. Nothing else is in scope here until Stage 4 is locked.

## Handoff discipline (standing rule, every session)

Maintain a single running file, **`HANDOFF.md`** (repo root), and keep it current — update it
at every reasonable checkpoint: after a decision is made, after finishing a meaningful chunk of
work, before context is likely to be compacted, and always before ending a session. **Overwrite
it in place** — do not create a new dated file each time. It should always answer "if this chat
ended right now, what would the next session need to know": what changed, what was decided and
why, what's open/blocked, and anything time-sensitive (e.g. files sitting in an ephemeral
scratchpad path that need rescuing before they're cleaned up).

Older one-off dated docs (`AI_Continuation_Document-*.md`, `Session_Writeup_*.md`) are historical
archive from before this rule existed — never delete them, but `HANDOFF.md` is the current
source of truth and should be read first.

## Hard rules — read before writing any code

1. **NO ML detection models.** No YOLO, RT-DETR, Relationformer, U-Net, Siamese, or any
   trained-from-scratch detector/classifier architecture. Substitutes are **local VLMs, LLMs,
   and OCR models only.**
   - The only exception: if the fine-tuned VLM can't reach the pass-bar threshold (set in
     Phase 3.6), a dedicated detector may re-enter scope for Stage 4 specifically. This is a
     documented Plan B, not a default.
2. **Do not touch offline stages.** Anything already pure Python / OpenCV / NetworkX / regex
   in the real agent is out of scope. Not our problem in this repo.
3. **One shared base VLM, not five.** Stage 4's winner is fine-tuned ONCE for domain adaptation
   (task-neutral — symbol shapes, tag formats, drawing conventions), then a **separate LoRA
   adapter** is trained on top for the detection task specifically. The domain base stays clean
   so it can be reused later at other stages via prompts alone.
4. **Real data judges. Always.** Gupta and PID2Graph are real; Kaggle is synthetic. Synthetic
   is fine-tuning volume / typing-only proxy, never the pass/fail bar for detection.
5. **Gupta is class-agnostic.** Every box is labeled "Symbol" — no type. This is why Stage 4
   uses a TWO-PART metric: detection scored on Gupta (real), typing scored on Kaggle (synthetic).
   Never average these into one number. Always report both, with Kaggle's ontology-coverage %
   next to the typing score. **⚠️ Open question (see `Agent_Pipeline_Facts.md` §3, flag 3):**
   the agent's ontology is runtime/per-tenant, not fixed — "coverage %" needs a defined
   reference ontology (e.g. an illustrative/representative set) before this can be computed
   honestly. Not yet resolved.
6. **Detection pass bar is mAP@0.5 or F1 — never recall alone.** A model that emits thousands
   of boxes scores near-perfect recall with garbage precision. Precision must be part of the bar.
7. **Test-set discipline.** The 20 Gupta test sheets are frozen in `test_ids.json` the moment
   it's built. Zero overlap with train, asserted every time, checked before every training run.
8. **No MLflow.** Use `results.csv` + `experiments/stage4/v*.md` instead — lighter weight, same
   job. See schema below.

## Read first, in this order

0. `HANDOFF.md` — the current running state of work, if it exists. Read this FIRST, before
   anything else in this list — it supersedes the older dated continuation docs as the source of
   truth for "what's going on right now."
1. `PID_Local_Substitution_Spec.md` — why this project exists, the substitution rule, the
   shared-base-VLM strategy (§5), the three VLM candidates (§5, Qwen3-VL / InternVL3 / Molmo)
2. `Stage4_Benchmarking_Checklist.md` — the actual execution plan, phase by phase, with a
   confirmation for every task. This is the authoritative to-do list.
3. `Stage4_Checklist_Status.md` — what's already done and reusable from prior work, vs. what's
   still open. Check this before starting any phase so you don't redo finished work.
4. `Agent_Pipeline_Facts.md` — **code-verified** facts about the real agent's Stage 3 tiling
   scheme, Stage 4 output schema, and entity ontology, pulled directly from
   `pnid-intelligence-agent` source. Three flags in here matter a lot: confidence and bbox are
   nested under `provenance.*` (not top-level), and the entity ontology is runtime/per-tenant,
   not a fixed enum — don't assume a flat/simple schema without checking this file first.
5. `base.md` — headline scores once runs exist.
6. `results.csv` — every run logged, one row each.

## Where we are right now

Phase 0/1 boundary. Roughly:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TomGeorge45/pid-opensrc-substitution](https://github.com/TomGeorge45/pid-opensrc-substitution) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
