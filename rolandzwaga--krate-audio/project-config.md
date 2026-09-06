---
trigger: always_on
description: ﻿# CLAUDE.md - VST Plugin Development Guidelines
---

﻿# CLAUDE.md - VST Plugin Development Guidelines

This file provides guidance for AI assistants working on this VST3 plugin project. All code contributions must comply with the project constitution at `.specify/memory/constitution.md`.

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

## Project Overview

This is a **monorepo** for Krate Audio plugins, featuring:
- **KrateDSP**: Shared DSP library at `dsp/` (namespace: `Krate::DSP`)
- **Iterum**: Delay plugin at `plugins/iterum/`
- **Disrumpo**: Multi-band distortion plugin at `plugins/disrumpo/`
- **Ruinae**: Synthesizer plugin at `plugins/ruinae/`
- **Innexus**: Harmonic analysis/resynthesis instrument at `plugins/innexus/` (AU type: `aumu`)
- **Gradus**: Standalone step arpeggiator at `plugins/gradus/` (AU type: `aumu`) — extracted from Ruinae's arp section, shares parameter IDs 3000-3372
- **Membrum**: Physically-modelled drum synthesizer at `plugins/membrum/` (AU type: `aumu`)
- **Seraphis**: Spectral-organism synthesizer at `plugins/seraphis/` (AU type: `aumu`, subtype `Srph`) — the current active-development plugin
- **Shared plugin infrastructure** at `plugins/shared/` (presets, UI components, MIDI, platform)
- **Steinberg VST3 SDK** (not JUCE or other frameworks)
- **VSTGUI** for user interface
- **Modern C++20**
- **CMake 3.20+** build system

### Monorepo Structure

The current roster is whatever lives under `plugins/` — **trust the filesystem, not a hand-maintained
tree here** (that is exactly where staleness accrues). Plugins today: `iterum`, `disrumpo`, `ruinae`,
`innexus`, `gradus`, `membrum`, `seraphis`, plus `shared`. The shared DSP library is `dsp/` (`Krate::DSP`, 5 layers
under `dsp/include/krate/dsp/{core,primitives,processors,systems,effects}/`).

- **Per-area detail** (skeleton, param-ID base, test target, pluginval path): the area `CLAUDE.md` leaf
  files (see below) and the generated maps under `specs/_architecture_/` (`repo-map.json`, layer/plugin
  reference docs).
- **Dev tooling** is in `tools/`; feature specs (numbered) in `specs/`; the VST3 SDK is vendored at
  `extern/vst3sdk/`. Other deps (pffft, Highway, dr_libs) are fetched/vendored — see External Dependencies.

## Per-Directory Context Files

This file holds cross-cutting rules. **Area-specific `CLAUDE.md` leaf files auto-load when you work in
their subtree** — read them for the concrete facts (skeleton, param-ID scheme, test target, pluginval path):

- [`dsp/CLAUDE.md`](dsp/CLAUDE.md) — layer architecture, ODR procedure, header-only/SIMD conventions
- [`plugins/iterum/CLAUDE.md`](plugins/iterum/CLAUDE.md) · [`disrumpo`](plugins/disrumpo/CLAUDE.md) · [`ruinae`](plugins/ruinae/CLAUDE.md) · [`innexus`](plugins/innexus/CLAUDE.md) · [`gradus`](plugins/gradus/CLAUDE.md) · [`membrum`](plugins/membrum/CLAUDE.md) · [`seraphis`](plugins/seraphis/CLAUDE.md)

## Critical Rules (Non-Negotiable)

### No Amending Commits

**NEVER use `git commit --amend`.** Always create a new commit. Amending is ONLY allowed when the user explicitly asks for it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rolandzwaga/krate-audio](https://github.com/rolandzwaga/krate-audio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
