---
trigger: always_on
description: > Read this first. It is the single source of truth for **what we are building, why, the
---

# AGENTS.md — TRINITY Replication

> Read this first. It is the single source of truth for **what we are building, why, the
> rules of the environment, and how every contributor (human or AI agent) must record
> their work.** If you are an AI agent picking up this repo, follow this file exactly.

---

## 1. The goal

Replicate the results of the paper **"TRINITY: An Evolved LLM Coordinator"**
(Xu, Sun, Schwendeman, Nielsen, Cetin, Tang — Sakana AI, ICLR 2026; arXiv:2512.04695v3)
using **open-source models** as the coordinated LLM pool instead of the paper's original
models.

TRINITY is a lightweight **coordinator** that orchestrates collaboration among several large
language models without touching their weights. It is made of two small pieces:

- a **compact language model (~0.6B parameters)** whose hidden states give a rich
  contextual representation of the query, and
- a **lightweight head (~10K parameters)** that turns that representation into a decision.

The coordinator works **over multiple turns**. At each turn it picks one LLM from the pool
and assigns it one of three roles — **Thinker, Worker, or Verifier** — so the heavy skills
live in the big models while the coordinator only learns *who should do what, when*. The head
is trained with a **separable CMA-ES** evolutionary strategy (no gradients through the big
models, no RL), which the paper shows beats RL, imitation learning, and random search under
high-dimensional, tight-budget conditions.

**Headline claim we are chasing:** TRINITY consistently beats every individual model and prior
coordination/routing methods on coding, math, reasoning, and domain knowledge, generalizes to
out-of-distribution tasks, and set a record of **86.2% on LiveCodeBench**.

### What "replication" means here

Our LLM pool is different from the paper's, so **absolute scores will differ**. Success is
defined by the paper's **relative** claims holding on our pool:

1. **Coordinated TRINITY > the best single model in the pool** on the in-distribution
   benchmark suite.
2. **TRINITY > random / static routing** baselines (same pool, same budget).
3. **sep-CMA-ES > RL / imitation / random-search** for training the head (reproduce the
   optimizer comparison).
4. The trained coordinator **generalizes to held-out / OOD tasks**.

The detailed, paper-grounded implementation spec lives in **`docs/SPEC.md`** (generated from a
careful multi-pass read of the paper). When SPEC.md and this file disagree on a number, SPEC.md
wins for implementation detail; this file owns the *intent*.

---

## 2. The model pool (open-source, via Fireworks)

The three coordinated LLMs are served through the **Fireworks AI** OpenAI-compatible API:

| Slot | Fireworks model ID                               | Short name      |
| ---- | ------------------------------------------------ | --------------- |
| A    | `accounts/fireworks/models/deepseek-v4-pro`      | `deepseek-v4-pro` |
| B    | `accounts/fireworks/models/glm-5p2`              | `glm-5p2`       |
| C    | `accounts/fireworks/models/kimi-k2p6`            | `kimi-k2p6`     |

All three were confirmed reachable with the project API key. The compact ~0.6B coordinator
encoder runs **locally on our own GPU** (see below), not on Fireworks.

---

## 3. The compute environment

- **Remote GPU box:** SSH alias `trinity-gpu` (configured in `~/.ssh/config`).
  - Hardware seen on login: **NVIDIA H200 NVL, 143 GB**, on **GPU index 5**.
  - **We are allocated GPU 5 ONLY.** Every remote process that touches CUDA **must** set
    `CUDA_VISIBLE_DEVICES=5`. Do not run anything on GPUs 0–4 or 6+. The helper scripts in
    `scripts/` enforce this; if you bypass them, set the variable yourself.
- **Local box:** orchestration, code, git. The 0.6B encoder + CMA-ES loop are meant to run on
  the remote H200; LLM calls go out to Fireworks over HTTP.

---

## 4. Security rules — non-negotiable

**No secret ever enters this repository or any commit.** This repo is private, but treat it as
if it were public.

- The **SSH private key** lives at `~/.ssh/trinity_gpu` (perms `600`), referenced only by the
  `trinity-gpu` host alias in `~/.ssh/config`. It is never copied into the project tree.
- The API key may live in repo-root `secrets.env` (ignored by git) or in
  `~/.config/trinity/secrets.env` (perms `600`). Code auto-loads either file and reads the
  provider key from the environment.
- Before running anything: make sure one of those files exists, or set the env var in the
  current process some other way.
- `.gitignore` blocks `.env`, `*.key`, `*.pem`, `*trinity_gpu*`, `secrets.env`, `*token*`, etc.
  as defense-in-depth. **Run `git diff --cached` before every commit** and abort if any secret
  appears.
- Never paste the key into code, logs, prompts, commit messages, issue text, or screenshots.

---

## 5. Repository layout

```
trinity/
├── AGENTS.md              # this file — goal, rules, logging protocol
├── README.md              # quickstart
├── docs/
│   ├── SPEC.md            # implementation spec distilled from the paper
│   ├── PAPER_NOTES.md     # raw section-by-section extraction of the paper
│   ├── JOURNAL.md         # ★ running log of mistakes, findings, decisions (see §6)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mini-router/minirouter](https://github.com/mini-router/minirouter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
