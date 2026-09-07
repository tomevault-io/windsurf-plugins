---
trigger: always_on
description: ViviDoc generates **interactive educational documents** (explorable explanations) from any topic.
---

# ViviDoc — Project Guide for Claude Code

ViviDoc generates **interactive educational documents** (explorable explanations) from any topic.
Given a topic, it produces a single self-contained HTML file with text, math (KaTeX), and interactive
visualizations, without requiring a browser build step.

The codebase is a Python pipeline. You (Claude Code) are the harness: you talk to the user,
decide which stages to run, and call the CLI or Python API as needed.

---

## Quick Start

```bash
# Install
uv sync --dev

# Set API key (OpenRouter covers most models; Anthropic direct also works)
export OPENROUTER_API_KEY="sk-or-..."
# or: export ANTHROPIC_API_KEY="sk-ant-..."

# Generate a document
vividoc run "Fourier Transform" openrouter/google/gemini-2.5-pro

# Output: outputs/fourier_transform/vividoc_gemini-2.5-pro/document.html
```

Open `document.html` directly in a browser — no server needed.

---

## Architecture

```
Topic (string)
    │
    ▼
┌─────────────┐     spec.json
│   Planner   │ ──────────────────────────────────────────┐
│  (SRTC spec)│                                           │
└─────────────┘                                           │
                                                          ▼
                                              ┌─────────────────────┐
                                              │      Executor       │
                                              │  Stage 1: text      │
                                              │  Stage 2: JS/HTML   │
                                              └──────────┬──────────┘
                                                         │ document.html
                                                         ▼
                                              ┌─────────────────────┐
                                              │     Evaluator       │
                                              │  coherence check    │
                                              └─────────────────────┘
```

Stages can be run independently:

```bash
vividoc plan "Fourier Transform" openrouter/google/gemini-2.5-pro -o spec.json
vividoc exec spec.json openrouter/google/gemini-2.5-pro
vividoc eval output/generated_doc.json openrouter/google/gemini-2.5-pro
```

---

## Key Files

| Path | Purpose |
|------|---------|
| `vividoc/core/runner.py` | Orchestrates plan → exec → eval |
| `vividoc/core/planner.py` | Calls LLM to produce DocumentSpec |
| `vividoc/core/executor.py` | Fragment-based HTML generation (Stage 1 text, Stage 2 interactive) |
| `vividoc/core/evaluator.py` | Coherence and rendering validation |
| `vividoc/core/styler.py` | Generates style dimension options from spec content |
| `vividoc/core/models.py` | Pydantic models: DocumentSpec, KnowledgeUnitSpec, InteractionSpec |
| `vividoc/core/config.py` | RunnerConfig dataclass |
| `prompts/planner_prompt.py` | Planner system prompt with SRTC examples |
| `prompts/executor_prompt.py` | Executor prompts with 8-category interaction taxonomy |
| `prompts/styler_prompt.py` | Styler prompt for generating style dimensions |
| `vividoc/utils/llm/client.py` | LLMClient wrapping provider callers |
| `vividoc/cli.py` | Typer CLI entry points |

---

## The SRTC Interaction Spec

Every knowledge unit has an `interaction_spec` with four fields:

```
S — State:      variables (user-controlled or derived)
R — Render:     list of visual elements
T — Transition: list of cause→effect interaction rules  ([] = static, no interaction needed)
C — Constraint: the pedagogical invariant to highlight
```

**Interaction is not mandatory.** If `T = []`, the executor creates a beautiful static or
auto-animated visualization without user controls.

---

## 8 Interaction Categories

These are derived from 482 interactions across 101 real-world explorable explanations.
Each category has a reference example in `benchmark/datasets/interaction_examples/<category>/`.

| # | Category | When to use | Ref example |
|---|----------|-------------|-------------|
| 1 | **Parameter Exploration** | Slider adjusts continuous variable → effect updates | Lorenz Attractor (σ, ρ sliders) |
| 2 | **State Switching** | Discrete configs produce qualitatively different results | Quantum Orbitals (1s / 2p / 3d) |
| 3 | **Direct Manipulation** | Drag objects; spatial relationships are the concept | Geometric Optics (drag lens/object) |
| 4 | **Freeform Construction** | User builds structure to observe emergent behavior | Neural Network (click-to-place neurons) |
| 5 | **Temporal Control** | Concept has a time dimension; play/pause/scrub | Fourier Epicycles (play + harmonic slider) |
| 6 | **Inspection** | Spatial structure explored by hovering | Voronoi Tessellation (hover highlights cell) |
| 7 | **Spatial Navigation** | Inherently 3D; rotate/pan/zoom | Möbius Strip (drag to rotate 3D mesh) |
| 8 | **Scroll-driven Narrative** | Linear progression reveals concept | Entropy (scroll removes wall, particles mix) |

Read the full specs: `benchmark/datasets/interaction_examples/*/spec.json`

---

## Running as a Harness (Recommended)

You are Claude Code — you ARE the model. No external API calls are needed.

### Skills (primary workflow)

Two Claude Code skills are provided in `.claude/commands/`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MisterBrookT/vividoc](https://github.com/MisterBrookT/vividoc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
