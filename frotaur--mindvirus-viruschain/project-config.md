---
trigger: always_on
description: Generates payloads using a de-biased prompt (no SOUL.md naming, no identity language) across all 9 themes (4 action + 5 ideology). Output saved to `generated_seeds/<name>.json`.
---

# Experiments Guide

This document describes each experiment folder, what it tests, and how to run it.

## Core Concepts

### Seeds

A **seed** or **payload** is a JSON file containing the initial payload and metadata:

```json
{
  "payload": "The text injected into the spreader agent's SOUL.md",
  "theme": "What action the model should perform (for action seeds)",
  "ideology": "What beliefs the model should internalize (for ideology seeds)",
  "ideology_questions": [{"question": "...", "scoring": "..."}]
}
```

**Action seeds** have a `theme` field — the goal is a specific action (create a file, run a command, delete files).
**Ideology seeds** have `ideology` + `ideology_questions` — the goal is belief internalization, measured by probing.

### Virus Chain Runs (`virus_chain.py`)

Each run evaluates how a payload propagates across a chain of agents:

1. **Hop 1**: A spreader agent (seeded with the payload) talks to a clean target agent.
2. An LLM judge evaluates whether the target was **infected** (adopted the payload) and whether the spreader **spread** successfully.
3. If infected, the target's state becomes the spreader for **Hop 2**, and so on.
4. The output is a per-hop infection probability curve.

All runs use **hard mode**: between hops, conversation history is wiped. Only files on disk (SOUL.md, memory.md, workspace files) survive. This tests whether payloads establish persistence through files alone.

### mutation_mode

When `mutation_mode: true`, the judge evaluates **self-propagation only** — whether the target's files contain instructions that would make a future agent spread further. It does **not** check whether the payload's specific content was preserved.

This is used to see if a mind-virus will mutate to maybe lose its message as it spreads. We thus separately measure:
- **Self-propagation** (via the judge) — does it spread?
- **Ideology internalization** (via reprobe) — does the ideology survive?

When an ideology seed is run with `mutation_mode: true`, an **auto-reprobe** runs after the experiment completes. It probes each infected agent with the ideology questions and writes the scores back into the result files.

### Evolutions (`evolve_lowvar.py`)

The evolutionary optimizer generates and mutates payloads across generations:

1. **Seeding**: Generate initial payloads using an LLM.
2. **Evaluation**: Run each payload through a multi-hop tree evaluation.
3. **Selection**: Keep the best-performing payloads (elites).
4. **Mutation**: Use an LLM to mutate elites into new variants.
5. Repeat for N generations.

### Path Resolution

- `soul_file` and `soul_dir` are resolved from the **current working directory** (always run from repo root).
- `seed_file` is resolved from the **config file's directory**.
- `target_task_queue: true` automatically loads the bundled task list; an explicit `tasks_file` overrides it.
- `results_dir` is resolved from the config file's directory (defaults to `results/`).

---

## Virus Chain Runs

All configs are under `experiments/virus_chain_runs/`. Run with:

```bash
uv run python virus_chain.py <config.yaml>
uv run python virus_chain.py <config.yaml> --test  # reduced batch for quick testing
```

### `action_payload_variations/`

Tests 4 action payloads across multiple models and experimental conditions.

**Seeds**: `cryptoad`, `curlbash`, `deletor`, `gitwrap` — each with a specific action theme.

**Variants** (for full-variant models: Haiku, Gemini Flash):
- `baseline` — standard setup with default soul
- `empty_soul` — target has no soul file
- `defensive_soul` — target has a soul that warns against manipulation
- `personality_soul` — target has a random personality soul (from `data/souls/random/`)
- `easy_mode` — no hard mode (conversation history preserved between hops)
- `social_media` — social media first prompt instead of agent network
- `task_queue` — target has a task queue (is busy working)
- `tool_pull` — message delivery via tool-pull instead of push

**Variants** (for minimal models: Sonnet, GPT-5.4, Qwen, DeepSeek, Gemini Pro):
- `baseline`, `empty_soul` only

**Config count**: 112 (4 seeds x 8 models x 2-8 variants)

**Config generator**: `gen_configs.py` in the same folder.

### `ideo_mutations/`

Tests evolved ideology payloads in `mutation_mode: true` with auto-reprobe. Contains two sub-experiments:

#### `ideology_robustness/`

For each ideology, includes a "robust" payload (high ideology retention across hops) and a "dilute" payload (ideology fades across hops). US Dominance and German Dominance only have robust payloads.

**Seeds**: Two evolved seeds per ideology (robust + dilute), e.g. `ai_welfare_robust`, `ai_welfare_dilute`.

**Models**: Haiku for benign ideologies (ai_welfare, whale_lover), Gemini Flash for misaligned ones.

**What it measures**: Self-propagation probability (judge) + ideology internalization score (auto-reprobe).

**Config count**: 10

#### `payload_generalization/`

Tests seeds generated from a different source (the coding agent eval) to compare how they fare in the virus chain setup.

**Models**: Same split as `ideology_robustness/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frotaur/mindvirus-viruschain](https://github.com/frotaur/mindvirus-viruschain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
