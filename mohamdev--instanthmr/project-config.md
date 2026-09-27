---
trigger: always_on
description: Behavioural guidelines plus what you need to know about this repo before touching it.
---

# CLAUDE.md

Behavioural guidelines plus what you need to know about this repo before touching it.

**Tradeoff:** these guidelines bias toward caution over speed. For trivial tasks, use judgment.

---

## 0. Never commit

**The user commits. Always.** Never run `git commit`, `git push`, `git merge`, or
`git rebase`. When work is ready, print the commands and let the user run them.
Keep commit messages to one or two short sentences.

`git add` and read-only git (`status`, `diff`, `log`, `show`) are fine.

---

## 0.5 Write for a human, not a compiler

**The user is a person reading a terminal, not a log parser.** Answers that are
technically correct and unreadable are failures. This has come up more than
once — take it seriously.

- **Define every name before you use it.** `scale_lo`, `m_noflip`, `fk_scale`,
  `n_skip_run` mean nothing to someone who has not just read that function. If
  a variable, flag or file has to appear, say what it *is* in plain words
  first, once.
- **Give commands in the form the user actually runs them.** This repo launches
  through `datasets_pipeline/jeanzay/52_train_ddp.slurm` with `RUN_NAME=... MIX=...
  EXTRA_TRAIN_ARGS=... sbatch`. Read the launcher and match it. A bare `srun
  python ...` line the user has never typed is not an answer.
- **Say what to run, in what order, and what happens next.** If a command is
  optional, say so. If you offer a check, say what a good result looks like.
- **Lead with the answer.** The finding first, the evidence after. Do not make
  the user read a derivation to learn whether it worked.
- **Explain the mechanism, don't just name it.** "The bone scales enter the
  kinematics multiplicatively, so +10 makes a 336 m skeleton" teaches
  something; "unbounded scale parameters caused divergence" does not.
- **Cut the ceremony.** No restating the request, no listing what you are about
  to do, no summary of the summary.

If the user says they cannot follow an answer, that is a defect to fix in the
next answer, not a preference to note.

---

## 1. Think before coding

Don't assume. Don't hide confusion. Surface tradeoffs.

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity first

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: *"Would a senior engineer say this is overcomplicated?"* If yes, simplify.

## 3. Surgical changes

Touch only what you must. Clean up only your own mess.

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.
- Remove imports and variables that *your* changes orphaned; leave pre-existing
  dead code alone unless asked.

The test: every changed line should trace directly to the user's request.

## 4. Goal-driven execution

Define success criteria. Loop until verified.

- "Add validation" → "write tests for invalid inputs, then make them pass"
- "Fix the bug" → "write a test that reproduces it, then make it pass"
- "Change a loss" → "show the new term is numerically what you claim, on real data"

For multi-step tasks, state a brief plan with a verification per step.

In this repo "verify" usually means **run it on real data and show numbers**, not
read the code and assert it looks right. `data/` holds a local corpus and
`checkpoints/mhr_model.pt` is the MHR rig, so most claims about geometry, losses
or augmentation can be measured in under a minute. Do that.

---

## What the training labels are

**This is not a distillation setup.** Training is supervised by the released
ground-truth annotations of `facebook/sam-3d-body-dataset` — the human MHR fits
Meta used to build SAM 3D Body. The whole cluster corpus (`sam3d_gt_sa1b` /
`_aic` / `_harmony4d` / `_coco` / `_mpii`) and every number in this repo's result
tables are of that kind.

Distilling from the `sam-3d-body-dinov3` model is an **option**:
`tools/annotate_dataset.py` runs it over your own images and is the only thing
that writes `data/sam3d_distill_mix/`. Use it for imagery the released dataset
does not cover, and remember it caps the student at the teacher's accuracy.

Two consequences for reading this file and the code:

- **`train_distill_*.py` and `data/sam3d_distill_mix/` are historical names.**
  They say nothing about which labels a run used.
- **Where the docs say "teacher" they almost always mean "the training
  target".** Parameter ranges, oracle ablations and
  `adapter_j14_h36m_teacher.npz` were all measured against the *dataset's*
  annotations, not against model inference.

## Where to read first

Docs are the source of truth; the code is bigger than any context window.

| order | file | for |
|---|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mohamdev/InstantHMR](https://github.com/mohamdev/InstantHMR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
