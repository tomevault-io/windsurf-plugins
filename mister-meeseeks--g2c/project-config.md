---
trigger: always_on
description: Guidance for AI coding agents working in this repo.
---

# AGENTS.md

Guidance for AI coding agents working in this repo.

## What this repo is

*From Gradients to ChatGPT* — a self-study course modeled after *From NAND to Tetris*, building a tiny LLM stack from scalar autodiff up through a working chat assistant. The repo serves dual purposes:

1. **Course material** — syllabus and per-module lessons (`docs/`)
2. **Work product** — the student's evolving implementation (`g2c/`)

Users of the repo may either be students working on the course, or project maintainers developing or updating the course.  Unit indicated otherwise, assume you're interacting with a course student.

## Course structure

The course is presented in **two parts**, split at the Module 11/12 boundary. The parts are a presentation layer only — module numbers are load-bearing in `notebook.sh`, `data/work/moduleNN/`, test names, and doc cross-links, so **never renumber modules**.

| Part | Group | Modules |
| ---- | ----- | ------- |
| I — From gradients to a language model | Prerequisite review | 00 |
| I | Foundations | 01, 02, 03, 03B |
| I | Language | 04, 05, 06 |
| I | The transformer | 07, 08, 09, 09B, 10, 11 |
| II — From a language model to ChatGPT | Behavior shaping | 12, 13, 13B, 14, 15 |
| II | Assistant systems | 16, 16B, 17, 18, 19, 20 |

Part I ends at Module 11 rather than Module 10 because a model without decoding loops on greedy sampling; Module 11 is what makes the artifact something a student can show someone. Part I is framed as a genuine finish line.

Two lesson pages carry **deliberate exceptions to the lesson-page template** below, both marking the part boundary. Don't "fix" them back:

- `11-sampling.md` ends with a `## You've finished Part I` section *after* `## Deliverable checklist` — the last thing on the page, where a reader lands when the module is done.
- `12-scaling.md` has an `## Entering at Part II` section between the intro and `## Before you start`, documenting direct entry.

Part II is close to self-contained: it imports only `g2c/nn/`, `g2c/training/` (Modules 03/03B) and `g2c/sampling/` (Module 11), and its BaseLM/ProdLM paths don't need a student-trained model. Module 12 documents direct entry via `G2C_APPLY_SOLUTIONS=01-11`. If you change what Part II imports from Part I, update that section.

The groups are descriptive labels, not a numbered taxonomy — there is no "Phase I/II/III" scheme any more. **The part/group tables in `README.md`, `docs/index.md`, and `docs/syllabus.md` must stay identical**; the previous phase names silently drifted apart across those three files, which is how they became vestigial.

## Hard constraints

- **Runs on an M-series MacBook.** No cloud GPUs, no paid compute. Code, datasets, and model sizes must stay within what an M1/M2/M3/M4 with 16–64GB unified memory can execute.
- **From-scratch through the architecture.** Weeks 1–11 must not import a high-level abstraction for the concept under study. Don't use `torch.nn.MultiheadAttention` inside the attention module — the point is to build it. Using PyTorch tensor primitives, autograd (after week 1), and standard optimizers is fine when the concept under study is something else. Weeks 12–15 pivot to using the weights from a larger pretrained base model `BaseLM` . Weeks 16–20 pivot to `ProdLLM`: a local pretrained instruct Ollama backend sized to the student's machine.
- **Pedagogy over performance.** Code should be legible. Optimize for "every internal piece is understandable" over "this runs fastest." Performance work is its own later concern.

## Stack

- **Python 3.11+**
- **PyTorch with MPS backend** as primary
- **MLX** for inference-heavy stages where Apple-native performance matters
- **Jupyter notebooks** for exploration and visualization (in `notebooks/`)
- **Ollama / llama.cpp** for running pretrained open models in the capstone

Dependencies and build config live in `pyproject.toml`.

## Layout conventions

- `docs/modules/NN-name.md` — lesson + motivation + exercises + deliverable spec for module NN
- `docs/modules/NN-name/` — assets for that module (images, diagrams, supplementary files). Reference from the lesson with relative paths, e.g. `![](NN-name/summary.png)`.
- `g2c/<topic>/` — Python subpackage for that module's deliverable
- `g2c/notebook_extras/<topic>.py` — non-pedagogical notebook helpers (progress bars, matplotlib glue, run-orchestration wrappers) used by notebooks but not implemented by students
- `g2c/artifacts/models.py` — `save_model_artifact` / `load_model_artifact` implementing the durable model artifact convention from `docs/design/model-artifacts-and-tracks.md`. Use these rather than ad-hoc `torch.save` calls when persisting a trained model that downstream modules consume.
- `artifacts/models/<name>/` — saved model artifacts (`model.pt`, `config.json`, `manifest.json`); tokenizer is referenced by name in the manifest, not duplicated.
- `data/work/moduleNN/` — module-specific working files (rolling training checkpoints, hand-authored SFT/DPO datasets, sandbox directories for the tool/agent/capstone exercises). These are caches, not artifacts; safe to wipe to retrain.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mister-Meeseeks/g2c](https://github.com/Mister-Meeseeks/g2c) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
