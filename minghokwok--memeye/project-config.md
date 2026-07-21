---
trigger: always_on
description: This file contains Reflexion-specific instructions for adapting Mem-Gallery’s built-in Reflexion core to this benchmark’s Mem-Gallery-style dataset.
---

# AGENTS.md

## Scope
This file contains Reflexion-specific instructions for adapting Mem-Gallery’s built-in Reflexion core to this benchmark’s Mem-Gallery-style dataset.

Follow this file for all work in this subtree.

## Project Goal
Implement Reflexion on this project’s dataset, which follows the same general format as Mem-Gallery.

The target is a faithful benchmark adaptation of Mem-Gallery’s built-in Reflexion core, integrated into this repo’s local benchmark framework.

Do not adopt Mem-Gallery’s full benchmark scaffold as the main implementation base.

## Source of Truth

### Mem-Gallery Reflexion core is the source of truth for:
- Reflexion memory behavior
- reflection-related storage / recall / optimization behavior
- Reflexion-specific internal logic in the downloaded `memengine` code

Use the downloaded local Reflexion core under this subtree as the primary reference for core Reflexion behavior.

### This repo’s local benchmark code is the source of truth for:
- dataset parsing
- dialogue-to-memory conversion
- how image captions are injected into text-only memory
- how question images are represented at recall time
- the final QA loop structure
- prompt usage
- config style
- logging style
- output and runner conventions

Use:
- `Benchmark_Pipeline/benchmark/a_mem.py`
- `Benchmark_Pipeline/benchmark/methods.py`
- `Benchmark_Pipeline/benchmark/runner.py`
- local prompts already used in this repo
- local config conventions already used in this repo

Notes:
- In this setup, the downloaded Mem-Gallery code should provide the Reflexion core only.
- Do not rebuild Mem-Gallery’s full benchmark framework locally.
- There is no requirement to use Mem-Gallery’s `run_bench.py` as the main scaffold.
- Use `Benchmark_Pipeline/benchmark/a_mem.py` as the main local adapter reference for structure, benchmark integration pattern, and logging style.
- Reuse `Benchmark_Pipeline/benchmark/methods.py` and `Benchmark_Pipeline/benchmark/runner.py` if they are useful.
- Do not duplicate existing benchmark utilities if they already solve the needed problem cleanly.

## Non-Negotiable Rules
- Keep the adaptation thin.
- Build a local benchmark adapter around the downloaded Reflexion core.
- Do not rewrite Reflexion into MemEngine form beyond what has already been downloaded.
- Do not reimplement unrelated Mem-Gallery baselines.
- Do not redesign Reflexion core behavior.
- Do not silently substitute models, embedders, APIs, or retrieval components.
- If a required API, model, or config is missing, stop and ask.

## Critical Rule: Treat Reflexion as Text-Only for This Adaptation
For this benchmark adaptation, treat Reflexion as a text-only memory system unless the user explicitly asks for a new multimodal Reflexion extension.

Therefore:
- Do not add a native visual memory module.
- Do not add a separate visual embedding store.
- Do not add cross-modal retrieval.
- Do not redesign Reflexion into a new multimodal memory architecture.

For this benchmark, images must be incorporated using the existing text-only adaptation strategy already used in this repo:
- use the provided image caption from the dataset
- append image information into the stored text
- append question-image caption text into the recall query when needed

This is a benchmark adaptation layer. Do not present Reflexion itself as natively multimodal unless explicitly implementing a new multimodal extension.

## How to Represent Images for Reflexion
When a dialogue turn contains an image, store it as text in the adapter layer, not as a native image memory object.

Append:
- `image_caption`

Recommended stored-text pattern:

    <dialogue text>
    image:
    image_caption: <caption>

At recall time, if a question includes an image, append the question image caption to the recall query in the same textual style.

Do this in the adapter layer. Do not push this logic deep into the downloaded Reflexion core unless there is no clean wrapper alternative.

## Wrapper Boundary
The wrapper is responsible for:
- loading this repo’s Mem-Gallery-style dialogue data
- converting each dialogue turn into the text form expected by Reflexion
- injecting image captions into stored text
- injecting question-image captions into recall queries
- passing retrieved Reflexion context into the local benchmark QA loop
- reusing local benchmark utilities when useful
- saving outputs in the local benchmark format

The downloaded Reflexion core remains responsible for:
- core Reflexion memory behavior
- recall / store / optimize logic already implemented in the downloaded Mem-Gallery core
- Reflexion-specific data flow inside that core

## Code Organization
Prefer clean separation between:
- downloaded Reflexion core code
- local benchmark adapter code
- local experiment utilities

Preferred approach:
- keep the downloaded Reflexion core minimally modified
- place dataset parsing and benchmark glue outside Reflexion core files
- use wrapper or adapter code whenever possible
- reuse `Benchmark_Pipeline/benchmark/methods.py` and `Benchmark_Pipeline/benchmark/runner.py` where practical
- use `Benchmark_Pipeline/benchmark/a_mem.py` as the main local reference for the expected adapter pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MinghoKwok/MemEye](https://github.com/MinghoKwok/MemEye) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
