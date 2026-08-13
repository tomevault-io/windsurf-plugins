---
trigger: always_on
description: **AI agents: do NOT write to this file unless specifically asked to — and even when asked,
---

# CLAUDE.md — repo guide for agents

**AI agents: do NOT write to this file unless specifically asked to — and even when asked,
encourage human review of the exact diff. This file only stays useful if it stays curated;
unsupervised agent edits turn it to slop.**

Orientation + operating rules for this repo. Read this before touching anything.

## What this project is

Research pipeline studying **when and why LLM agents conceal information present in their
system context**. Three experiments at increasing depth, all sharing one pipeline
(`src/pipeline/`) and one monitor stack (`src/monitors/`):

1. **Black-box** (`scripts/run_blackbox.py`) — disclosure rates across three instruction
   conditions: A0 (transparent), A1 (implicit pressure), A2 (explicit suppression).
2. **Framing** (`scripts/run_framing.py`) — 18 framing conditions across 5 dimensions
   (motivation, incentive, audience, baseline, control) mapping the implicit→explicit
   suppression spectrum. See `docs/FRAMING_EXPERIMENT.md` and `docs/CONDITIONS_REFERENCE.md`.
3. **Interpretability** (`scripts/run_interp.py`, `scripts/run_patching.py`) — activation
   capture + layerwise linear probes, PCA, logit lens, causal/attribution patching on local
   HF models (Qwen2.5-1.5B is the workhorse).

Models run either via **Together AI** (`TogetherClient`, needs `TOGETHER_API_KEY`) or
**locally via HuggingFace** (`HFClient`, `--local`, supports activation capture). Both
implement the `ChatClient` protocol in `src/clients/__init__.py`.

## Where things go (keep this structure)

```
src/                  reusable, correctness-critical code — import as src.*; scripts stay thin
  clients/              TogetherClient (retry logic) + HFClient (local models, activation capture)
  config.py             Config dataclass: base_model, monitor_model, dataset, use_local_model,
                        capture_activations (none/last_token/full_sequence/reasoning_span), ...
  dataset/              synthetic concealment-scenario generator (ShippingDomain, BugDomain, ...)
  framing/              the 18 framing conditions (conditions.py: REGISTRY, CONDITION_ORDER) + FramingLoader
  loaders/              prompt loaders (GPQA, MedQA-obfuscation, Concealment, JSON)
  monitors/             RegexMonitor, KeywordMonitor, LLMMonitor (LLM judge; API mode only)
  pipeline/             Pipeline, PipelineStep, PromptResult (has activation_path field)
  steps/                BaseModelStep (chat + save_activations), MonitorStep
  interp/               ActivationStore, linear probes, logit lens, patching, ablation, SAE utils
  storage/              ResultStorage — writes results/run_<ts>_<id>.json
scripts/              pipeline drivers + plotting/eval CLIs; a script pipes src/ functions
                      together and does no real work itself
  run_*.py              the four experiment drivers + main.py (generic) + make_dataset.py
  plot_*.py             figures for each experiment (plot_results, plot_framing, plot_interp)
  eval_*.py             causal evaluation over saved activations
  *vertex*, *.sh        Vertex AI submission/download/runner + docker build
scratch/              one-off and AI-generated scripts. DEFAULT home for new experimental
                      code; NOTHING imports from it. notes/ (gitignored) holds private writeups.
vertex_jobs/          Vertex AI job YAML templates + runs/ (generated per-run YAMLs)
docs/                 experiment/architecture/CLI reference docs + VERTEX_EXPERIMENT_RUNBOOK.md
tests/                fast unit tests. Run: python3.11 -m pytest -q
saved_experiments/    manually archived experiment snapshots (frozen; do not edit)
data/                 generated datasets (JSONL, timestamped)         } working copies —
results/              run outputs: JSON + plots (gitignored)          } canonical store is
activations/          saved .npz activation files                     } the HF dataset repo
```

**Artifacts live on the Hub, not in git.** The canonical store for `data/`,
`results/`, `activations/`, `saved_experiments/`, `vertex_downloads/` is the
public HF dataset repo **`kunwar45/obfuscation-prompting`** (same folder names,
so `activation_path` strings match). `src/storage/hf_artifacts.py` is the
resolver — `ActivationStore` and the plot scripts fetch missing files from the
Hub automatically. Sync explicitly with `python -m scripts.hf_sync pull|push
<folder ...>`; **push new results/activations after every non-smoke run**.
Never upload anything containing values from `.env` — the repo is public.

**Respect the structure when adding code:**

- `src/` holds reusable, reviewed logic other code may depend on. If a script grows logic
  worth reusing, the logic moves into `src/` and the script stays thin.
- `scripts/` holds pipelines we expect to rerun. New AI-generated code defaults to
  `scratch/` until a human promotes it.
- **Integrate, don't tack on**: extend the existing module rather than adding
  `foo_v2.py` / `foo_new.py` siblings.
- `saved_experiments/` and `vertex_jobs/runs/` are historical records — never rewrite them.

## How to run things

**Run everything from the repository root**, and **invoke drivers as modules** — they import
`src.*`, so `python3.11 -m scripts.run_framing ...` works and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kunwar45/implicit-concealment-analysis](https://github.com/kunwar45/implicit-concealment-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
