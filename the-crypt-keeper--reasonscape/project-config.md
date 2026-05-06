---
trigger: always_on
description: ReasonScape evaluates reasoning-capable language models using information-theory principles and parametric testing.
---

# ReasonScape - LLM Evaluation System Quick Reference

ReasonScape evaluates reasoning-capable language models using information-theory principles and parametric testing.

## Evaluation System: r12

## Shell Commands

Never use `run_in_background: true` for commands where you need the output (tests, builds, etc.). Run them directly with an appropriate `timeout`.

## Environment Setup

**Always start from project root:**
```bash
cd /home/mike/ai/reasonscape
source venv/bin/activate
```

## Dataset Structure

Raw evaluation data lives in `data/r12/`:
- `data/r12/ModelName/evals.json` - Model-specific eval definitions
- `data/r12/ModelName/2025-*/` - Raw interview NDJSON files
- `data/r12.json` - Primary dataset configuration
- `data/pointsdb-r12.db` - DuckDB database

Datasets (research-objective cohorts) live in `data/`:
- `data/<name>.json` - Dataset configuration
- `data/pointsdb-<name>.db` - DuckDB database

**Common datasets:**
- `data/r12.json` - Primary r12 dataset

There is no single dataset that imports all raw evaluation data, so you'll need to understand what dataset you're working with.

## Discovery: Mandatory First Step

Before ANY analysis, discover what evaluations exist inside the selected dataset:

```bash
# 1. List all evaluations (models) with their eval_id, groups, tier coverage
python analyze.py evals data/<name>.json

# 2. List all tasks with their surfaces and projections
python analyze.py tasks data/<name>.json

# 3. Get model metadata (params, architecture, tokenizer)
python analyze.py modelinfo data/<name>.json --eval-id <hash>
```

**Filter discovery results:**
```bash
# Search by model name
python analyze.py evals data/r12.json --search qwen

# Filter by groups (AND logic)
python analyze.py evals data/r12.json --groups size:large,arch:moe
```

## Research Workflows: The Four P's

### Position - Ranking Models
"Which model is better?"

```bash
# Absolute overall ranking by aggregate score
python analyze.py scores data/<name>.json

# Relative task ranking with confidence intervals + Relative overall rank
python analyze.py cluster data/<name>.json --stack base_task --rank
```

### Profile - Characterizing Capabilities
"What can this model do?"

```bash
# Single model fingerprint
python analyze.py compare data/<name>.json --filters '{"eval_id": ["a1b2c3"]}'

# Capability boundaries across tasks
python analyze.py surface data/<name>.json \
  --filters '{"eval_id": ["a1b2c3"]}' \
  --split base_task \
  --output-dir research/<project>/

# Compare multiple models
python analyze.py surface data/<name>.json \
  --filters '{"eval_id": ["a1b2c3", "d4e5f6", "123abc"]}' \
  --split base_task \
  --output-dir research/<project>/
```

### Point - Diagnosing Failures
"Why does this fail?"

```bash
# Failure boundaries (single model, single task)
python analyze.py surface data/<name>.json \
  --filters '{"eval_id": ["a1b2c3"], "base_task": "arithmetic"}'

# Tokenization analysis
python analyze.py fft data/<name>.json \
  --filters '{"eval_id": ["a1b2c3"], "base_task": "arithmetic"}'

# Reasoning efficiency
python analyze.py compression data/<name>.json \
  --filters '{"eval_id": ["a1b2c3"], "base_task": "arithmetic"}' \
  --output-dir research/<project>/
```

### Probe - Examining Raw Traces
"What does failure look like?"

```bash
# Loop detection in truncations
python probe.py truncation data/<name>.json \
  --filters '{"eval_id": ["a1b2c3"], "base_task": "arithmetic"}' \
  --output research/<project>/loops.json
```

## Filter Syntax Reference

**Critical:** Use SINGLE quotes around JSON, DOUBLE quotes inside.

```bash
# Filter by eval_id (6-char hash identifying model+template+sampler)
--filters '{"eval_id": ["a1b2c3", "d4e5f6"]}'

# Filter by groups (tags)
--filters '{"groups": [["arch:moe", "size:large"]]}'

# Filter by tier difficulty
--filters '{"tiers": ["easy", "medium"]}'

# Filter by task
--filters '{"base_task": ["arithmetic", "shuffle"]}'

# Filter by surface (2D visualization)
--filters '{"surfaces": ["arith_len_x_depth"]}'

# Filter by projection (1D FFT analysis)
--filters '{"projections": ["arith_length_d0"]}'

# Combine filters (AND logic)
--filters '{"eval_id": ["a1b2c3"], "base_task": "arithmetic", "tiers": ["easy", "medium"]}'
```

**Groups filter syntax:**
- Single group: `'{"groups": [["arch:moe"]]}'` - matches ANY eval with arch:moe
- Multiple groups (OR): `'{"groups": [["arch:moe"], ["arch:dense"]]}'` - matches evals with EITHER tag
- Multiple groups (AND): `'{"groups": [["arch:moe", "size:large"]]}'` - matches evals with BOTH tags

## Common analyze.py Subcommands

```bash
# Discovery
python analyze.py evals <config>           # List evaluations
python analyze.py tasks <config>           # List tasks/surfaces/projections
python analyze.py modelinfo <config>       # Model metadata

# Position
python analyze.py scores <config>          # Aggregate ranking
python analyze.py cluster <config>         # Statistical ranking

# Profile
python analyze.py compare <config>         # Bar/radial chart fingerprint
python analyze.py surface <config>         # 3D capability landscapes
python analyze.py fft <config>             # Tokenization spectral analysis
python analyze.py compression <config>     # Reasoning efficiency


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-crypt-keeper/reasonscape](https://github.com/the-crypt-keeper/reasonscape) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
