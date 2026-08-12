---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**10xBench** is a comprehensive benchmark comparing how different large language models handle "vibe coding"—creating a fully functional website for [Przeprogramowani.pl](https://przeprogramowani.pl) in a single attempt without iterative refinement.

Each of four models (Claude Opus 4.6, GPT-5.3-Codex, Kimi K2.5, GLM-4.7) receives the same prompt and must generate a complete website implementation. Results are systematically evaluated against defined criteria and presented in an interactive dashboard.

## Quick Start Commands

### Development & Building

```bash
# Process results CSV files and generate results.json
npm run process-results

# Start development server (Astro) with results processing
npm run dev

# Build production site
npm run build

# Development from website directory (for Astro only)
cd website && npm run dev
cd website && npm run build
```

### Evaluation

```bash
# Run the evaluation skill to assess an implementation
/10x-score-attempts against <attempt-directory>

# Example:
/10x-score-attempts against claude opus attempt 1
```

## Project Structure

### Core Directories

- **`./scripts/`** — Data processing utilities
  - `process-results.ts` — TypeScript script that parses CSV evaluation files from each attempt and generates `results.json`

- **`./website/`** — Astro-based results dashboard
  - `src/pages/index.astro` — Main results overview page
  - `src/pages/benchmark.astro` — Benchmark prompt and criteria page
  - `src/components/ResultCard.tsx` — Summary cards showing attempt scores
  - `src/components/ResultsTable.tsx` — Interactive comparison table with sticky headers
  - `src/components/ModelAveragesCard.tsx` — Model family average summaries
  - `src/data/results.json` — Generated data file (created by `process-results.ts`)

- **`./eval-attempts/`** — Model implementations (one-shot attempts)
  - `claude-opus-attempt-{1,2,3}/` — Claude Opus 4.6 implementations
  - `gpt-codex-attempt-{1,2,3}/` — GPT-5.3-Codex implementations
  - `kimi-k2.5-attempt-{1,2,3}/` — Kimi K2.5 implementations
  - `glm-4.7-attempt-{1,2,3}/` — GLM-4.7 implementations
  - Each contains `eval-result.csv` with criterion-by-criterion scores

- **`./eval-results/`** — Processed evaluation results
  - Contains `eval-result.csv` files from each attempt (symlinks to source)

> **Note:** Evaluation criteria and scoring methodology live in the companion repo [10x-bench-eval](https://github.com/przeprogramowani/10x-bench-eval).

## Data Processing Pipeline

The `scripts/process-results.ts` script orchestrates the results workflow:

1. **Discovers** all eval-attempts directories
2. **Parses** `eval-result.csv` from each attempt
3. **Extracts** model name and attempt number from directory name
4. **Calculates** total scores and percentages (excluding "Task completion time")
5. **Aggregates** averages by model family
6. **Generates** `website/src/data/results.json`

### CSV Format Support

The script handles two CSV formats:

- **New format** (glm-4.7, gpt-codex): `Criterion,Score,Max,Notes`
- **Legacy format** (some attempts): `Criterion,Score,Notes` (assumes Max=1)

Both are parsed correctly and normalized to the same structure.

### Task Completion Time Format

The "Task completion time" row in eval-results CSV must use the standardized format `Xmin Ys` (e.g. `5min 46s`, `17min 3s`). Use `N/A` when time was not recorded. The format must be consistent in both the Score and Notes columns:

```csv
Task completion time,5min 46s,N/A,5min 46s
Task completion time,N/A,N/A,N/A
```

The `ResultsTable.tsx` component relies on this exact format to parse and display times.

### Test Run Format

The "Test run" row records when the evaluation was performed. It uses the format `D.MM.YYYY HH:MM` in both the Score and Notes columns, with `N/A` for Max:

```csv
Test run,9.02.2026 19:20,N/A,9.02.2026 19:20
```

Use `N/A` in Score and Notes when the test run date/time was not recorded. Every eval-results CSV should include this row immediately after "Task completion time".

## Results Dashboard (Astro Site)

### Pages

**`/` (Results Overview)**
- Displays all attempts sorted by percentage (best to worst)
- ResultCard components show summary stats with progress bars
- Color-coded scores: red (0-33%), yellow (33-66%), green (66%+)
- ModelAveragesCard shows family averages at the bottom

**`/benchmark`**
- Full-width display of the benchmark prompt (`benchmark/prompt.md`)
- Full-width display of evaluation criteria (`benchmark/criteria.md`)
- Both rendered as HTML from markdown

### Interactive Features

- **ResultsTable** (on index page)
  - Sticky header for scrolling context
  - Frozen first column (criterion names) for horizontal scrolling
  - Clickable notes that expand inline
  - Responsive design for mobile

- **Progress bars** in ResultCard components color-coded by performance tier

## Technology Stack

### Website (Astro Framework)

- **Astro 5.x** — Static site generator
- **React 19** — UI components (ResultCard, ResultsTable, ModelAveragesCard)
- **Tailwind CSS 3.x** — Styling (v4 incompatible in this environment)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [przeprogramowani/10x-bench](https://github.com/przeprogramowani/10x-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
