---
trigger: always_on
description: Instructions for AI coding agents (Claude Code, Copilot, Cursor, Codex, etc.) working inside this repository. Read this file before answering questions about the repo or editing its contents.
---

# AGENTS.md

Instructions for AI coding agents (Claude Code, Copilot, Cursor, Codex, etc.) working inside this repository. Read this file before answering questions about the repo or editing its contents.


## Repository purpose

CUDA-Q Academic is a collection of Jupyter notebooks that teach quantum computing with [CUDA-Q](https://developer.nvidia.com/cuda-q). It is organized as ten self-contained lesson modules, each of which lives in a top-level folder. Human-facing overview: [README.md](README.md). Hosted curriculum and launch page: https://nvidia.github.io/cuda-q-academic/learningpath.html.

The authoritative machine-readable content catalog in this repo is [curriculum.json](curriculum.json). It is the source of truth for lesson discovery, track membership, prerequisites, difficulty, keywords, summaries, and the live visualization-gallery widget inventory.

## Scope of this repo vs. CUDA-Q documentation  — **read this before answering programming questions**

This repository is **educational material** (lessons, exercises, worked solutions, widgets). It is **not** the CUDA-Q product documentation and is **not** the canonical source for CUDA-Q API reference, language specification, install instructions, backend configuration, or runnable code examples.

When a user asks an API, syntax, install, backend, performance, or "how do I do X in CUDA-Q?" question, route them to the upstream CUDA-Q docs and docs repo rather than answering from notebooks in this repo:

- **Documentation site (primary):** https://nvidia.github.io/cuda-quantum/latest/
  - API reference (Python): https://nvidia.github.io/cuda-quantum/latest/api/languages/python_api.html
  - API reference (C++): https://nvidia.github.io/cuda-quantum/latest/api/languages/cpp_api.html
  - Examples gallery: https://nvidia.github.io/cuda-quantum/latest/examples/examples.html
  - Backends and targets: https://nvidia.github.io/cuda-quantum/latest/using/backends/backends.html
  - Install / quick start: https://nvidia.github.io/cuda-quantum/latest/using/quick_start.html
- **Documentation source repo (secondary):** https://github.com/NVIDIA/cuda-quantum
  - Example source: https://github.com/NVIDIA/cuda-quantum/tree/main/docs/sphinx/examples
  - Snippets used in docs: https://github.com/NVIDIA/cuda-quantum/tree/main/docs/sphinx/snippets
  - Python package source: https://github.com/NVIDIA/cuda-quantum/tree/main/python

Use this repo (CUDA-Q Academic) when the question is about *learning order, lesson content, exercises, prerequisites, or pedagogy*. Use the upstream docs and docs repo when the question is about *the CUDA-Q library itself*. When in doubt, link both: cite the relevant lesson here for context and the upstream docs page for the canonical reference.

## Content inventory  — **primary behavioral instruction**

When a user asks what content exists, which track or lesson covers a topic, what the prerequisites or difficulty are, which widgets are live, or where to deep-link them, start with [curriculum.json](curriculum.json).

1. Use `track_order` plus `tracks` to enumerate the ten learning paths. A track's ordered `lesson_ids` list is the canonical lesson sequence for that path.
2. Use `lessons` to resolve a lesson's title, links, prerequisites, difficulty, keywords, summary, and cross-track membership. Shared notebooks appear once and may be referenced by multiple tracks.
3. Use `widget_gallery_order` plus `widgets` to enumerate the live Visualization Gallery. Widget source files may live on the `widgets-as-html` branch even when they are absent from `main`; rely on each widget's `source_url` and `source_repo_path`.
4. When a lesson has `source_kind == "external_notebook"`, trust the catalog link instead of searching for a local file in this repo.
5. When `curriculum.json` and a notebook intro disagree, treat that as drift and update both in the same change.


## Inspecting a notebook directly  — **secondary detail source**

When a user needs notebook-level detail beyond what is in `curriculum.json` (for example the full "What You Will Do" list, CUDA-Q APIs used, or GPU requirement), inspect the notebook itself:

1. Resolve the lesson in [curriculum.json](curriculum.json) first when possible, then locate the notebook path (see **Repository layout** below).
2. For each `.ipynb` in that folder, parse the JSON and find the **first cell whose `cell_type == "markdown"`**. (Most notebooks open with an SPDX license code cell at index 0; the intro markdown cell is therefore at index 1. A few notebooks — e.g. `ai-for-quantum/01_compiling_unitaries_diffusion.ipynb` — place the markdown at index 0. Always use *first markdown cell*, not a fixed index.)
3. That cell follows a fixed schema defined by [notebook_template.ipynb](notebook_template.ipynb):

   ```markdown
   # <Title> — <Module>: <Subtitle>

   **What You Will Do:**
   * <learning objective>
   * ...

   **Prerequisites:**
   * ...

   **Key Terminology:**
   * ...

   **CUDA-Q Syntax:**
   * [`<api>`](<docs_url>) — <one-line description>
   * ...

   **Solutions:** [`solutions/<notebook>_solutions.ipynb`](solutions/...)
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/cuda-q-academic](https://github.com/NVIDIA/cuda-q-academic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
