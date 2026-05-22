---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Qlatt is a WebAudio Klatt formant synthesizer with TTS frontend. It implements the Klatt 1980 synthesizer model using WASM-backed AudioWorklets for DSP processing, driven by a declarative YAML-based configuration system.

## Core Principles

### 1. Explainability Is Not Optional

This is an *explainable* synthesizer. Every decision the system makes — which phoneme target was selected, which rules fired, why F0 is 142 Hz at this instant, why B1 widened here — must be traceable back to a specific rule, a specific paper, and a specific justification.

This is enforced through three interlocking systems:

**Provenance** (`src/provenance.ts`, `src/tts-frontend-provenance.ts`):
- `ProvenanceCollector` records a `DecisionRecord` for every pipeline decision
- Each record has: `stage`, `type`, `subject`, `reason`, `citations[]`, `parents[]`
- Records form a DAG: inventory selection → rule match → rule rewrite, linked by parent IDs
- The rule trace captures every match and rewrite event with the originating token ID

**Citations on rules** (every rule in `public/rules/frontends/<frontend-id>/phases/*.yaml`):
- Every rule has a `citations:` array linking to the paper or source that justifies it
- These citations flow into provenance records automatically via `RULE_CITATIONS` map
- A rule without citations is a bug. If you write a rule, cite its source.

**Tags on rule applications** (the `tag:` field on apply/dispatch entries):
- Tags like `stress`, `consonant_position`, `boundary`, `f0_declination` label what *kind* of modification a rule made
- These feed into diagnostics and provenance, making it possible to ask "why is this vowel short?" and get "duration was multiplied by 0.7 (tag: consonant_position, citation: Klatt 1976 Table III)"

**What this means for you as an agent:**
- When you write a new rule: include `citations:` with the paper reference. No citation = do not commit.
- When you add a rule application: include a `tag:` that describes the linguistic motivation.
- When you add a new pipeline stage: integrate with `ProvenanceCollector` — emit decision records for every non-trivial choice.
- When you modify semantics.yaml realize rules: comment the formula source. `# Fant 1960 Table 2.34-1` is the minimum.
- When you write Rust DSP code: cite the paper in a doc comment at the top of the module. See `crates/aerodynamic-model/src/lib.rs` for the pattern.
- Never introduce a magic number without a citation. If you can't cite it, label it `# engineering estimate` so we know it needs a real source later.

### 2. Always Cite Your Work

When we write code from a paper, cite that paper in the code. This is a specific instance of Principle 1.

The paper library lives in `papers/`. Each paper has a `notes.md` with implementation-focused extractions (equations, parameter tables, algorithms). Before implementing anything from a paper, read `papers/<Author_Year_ShortTitle>/notes.md` first — the extraction work is already done.

### 3. The Diagnostics System

`src/diagnostics.ts` provides runtime diagnostic logging (info/warn/error) with subscriber support. This is separate from provenance — diagnostics are for runtime observations ("warning: F0 clamped to floor"), while provenance is for decision tracing ("F0 set to 142 Hz because rule f0_declination matched, citation: O'Shaughnessy 1976").

When you add new processing stages or error conditions, emit diagnostics. When a parameter is clamped, defaulted, or falls back — that's a diagnostic event.

### 4. Script Repetitive Inspection And Migration Work

Do not rely on long shell or Node one-liners for non-trivial repo analysis, migration, or data inspection. If a task needs more than a simple command, write an actual script in `scripts/` (or another appropriate checked-in location) so the logic is readable, reusable, and reviewable.

**What this means for you as an agent:**
- For repeatable searches, inspections, or report generation: prefer a real script over an inline one-liner.
- For schema migrations or corpus analysis: write a dedicated script and run that script.
- Use one-liners only for genuinely trivial commands (`rg`, `Get-Content`, `git status`, short smoke checks).
- If a throwaway script is only needed temporarily, still make it a normal file first; delete it afterward if it should not stay in the repo.

## Build Commands

```bash
# Build WASM modules (required first)
pwsh -File build.ps1          # Windows
./build.sh                     # Unix

# Run dev server
npm run dev                    # Vite server at http://localhost:8000

# Golden tests
npm run test:golden            # Run golden comparison tests

# Build CMU dictionary
npm run build:dict

# Explain a phrase (provenance trace)
npm run explain -- "hello world"           # compact text output (~40 decisions)
npm run explain -- "hello world" --verbose  # all decisions
npm run explain -- "hello world" --format json --out report.json  # full JSON
npm run explain -- "hello world" --stage rules  # filter by pipeline stage
npm run explain -- "hello world" --why d000045  # trace ancestry of a decision

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ctoth/Qlatt](https://github.com/ctoth/Qlatt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
