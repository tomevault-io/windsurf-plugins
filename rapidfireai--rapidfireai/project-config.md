---
trigger: always_on
description: Operational instructions for **AI coding agents** (Claude Code, Cursor, Codex, GitHub Copilot, Windsurf, Aider, Junie, and similar) that are helping an end user install, configure, and run the `rapidfireai` Python package.
---

# RapidFire AI — Agent Install & Setup Guide

Operational instructions for **AI coding agents** (Claude Code, Cursor, Codex, GitHub Copilot, Windsurf, Aider, Junie, and similar) that are helping an end user install, configure, and run the `rapidfireai` Python package.

This file is **not** for rapidfireai contributors. If you are working *on* rapidfireai itself, stop and read the repo's root [`AGENTS.md`](https://github.com/RapidFireAI/rapidfireai/blob/main/AGENTS.md) and [`CONTRIBUTING.md`](https://github.com/RapidFireAI/rapidfireai/blob/main/CONTRIBUTING.md) instead.

## 1. Audience and authority

### Source-of-truth rule (read first)

This guide does **not** restate version-specific install commands, package versions, port numbers, or known-issue workarounds. Those live in the canonical [`README.md`](https://github.com/RapidFireAI/rapidfireai/blob/main/README.md) (sections **§Prerequisites**, **§Install and Get Started**, **§Troubleshooting**) and in the codebase, and they change between releases. **Whenever this guide and the README disagree on a specific command or version, trust the README.**

What this guide *does* provide that the README does not:

- A workflow decision tree (RAG vs fine-tuning vs post-training; OpenAI vs self-hosted; lite vs full).
- Trainer-type taxonomy for fine-tuning (`SFT` / `DPO` / `GRPO`).
- Safety rules for handling user secrets, GPU assumptions, and gated model access.

### Version awareness

After installing, run `rapidfireai --version` to see the live version. This guide assumes the **0.15+ API surface**. If the installed package differs significantly, prefer the canonical docs at <https://oss-docs.rapidfire.ai>.

Canonical raw URL of this file (for `WebFetch`): <https://raw.githubusercontent.com/RapidFireAI/rapidfireai/main/docs/AGENTS.md>.

---

## 2. Workflow decision tree

Pick a branch **before** running any install commands — the two workflows install different dependency sets that are not interchangeable.

- **User wants RAG / context-engineering evaluation** → use `Experiment(..., mode="evals")` and run the default `rapidfireai init` (evals dependencies are the default).
  - Generation/embedding via **OpenAI / Azure OpenAI / Anthropic** APIs → use `RFAPIModelConfig`. **No GPU strictly required** — viable on CPU-only machines.
  - Generation via **self-hosted Hugging Face** models → use `RFvLLMModelConfig`. **GPU required.** May require Hugging Face authentication for gated models.
- **User wants fine-tuning or post-training** → use `Experiment(..., mode="fit")` and run `rapidfireai init --train` (the training-only opt-in).
  - **SFT** (supervised fine-tuning, e.g., chat/QA tuning) → `trainer_type="SFT"`.
  - **DPO** (direct preference optimization, alignment from `chosen`/`rejected` pairs) → `trainer_type="DPO"`.
  - **GRPO** (group relative policy optimization, RL with reward functions) → `trainer_type="GRPO"`.

### Environment selectors

- **Remote / cloud machine** → an SSH port-forward is required to view the dashboard locally. The set of ports differs by workflow (smaller for fit, larger for evals because Jupyter and MLflow are also exposed). The current port set and the exact `ssh` command are in the README §Install — read them from there, do not memorize.
- **GPU issues** (CUDA absent, OOM, driver mismatch) → run `rapidfireai doctor` and act on its output. For OOM, switch the user to a *lite* tutorial variant (see §6).
- **Hugging Face permission issues** → confirm the user has run the README's HF auth step and has been granted access on the gated model's HF page. If access is blocked, suggest an open-license substitute (TinyLlama, Qwen-0.5B/3B) where licensing permits.

The full install command sequence and the exact set of port numbers are in the README, the authoritative install reference.

---

## 3. Setup order

Run the steps in the order below. The **commands** are in the README; the **decisions, ordering, and pitfalls** below are what you should add on top.

1. **Verify the user's environment matches README §Prerequisites *before* installing.** At minimum: (a) check the Python version (`python --version`); (b) for any **GPU-required workflow** — all fine-tuning (SFT/DPO/GRPO) and self-hosted RAG/eval — confirm GPU presence and CUDA via `nvidia-smi`. If GPU is required but absent, **stop and redirect**: the API-based RAG/eval workflow (with `RFAPIModelConfig`) does not need a GPU and is the only viable path on CPU-only hosts. Do not paper over a Python or GPU mismatch by trying to "fix" the system; surface the requirement to the user.
2. **Create and activate a virtual environment** *before* `pip install`. Do not assume the user is already in one.
3. **Install** the package as documented in README §Install. The exact `pip install` line lives there.
4. **Verify** the install: `rapidfireai --version` should return a version string. If it does not, stop and diagnose; do not proceed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RapidFireAI/rapidfireai](https://github.com/RapidFireAI/rapidfireai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
