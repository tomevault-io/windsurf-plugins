---
trigger: always_on
description: PAW compiles natural language specifications into tiny neural functions that run locally. Each function takes a single text input and returns a single text output. Use it when you need fuzzy text processing — classification, extraction, format repair, search, triage — that regex can't handle but a full LLM is overkill for.
---

# ProgramAsWeights (PAW)

PAW compiles natural language specifications into tiny neural functions that run locally. Each function takes a single text input and returns a single text output. Use it when you need fuzzy text processing — classification, extraction, format repair, search, triage — that regex can't handle but a full LLM is overkill for.

Website: https://programasweights.com
Full documentation: https://programasweights.readthedocs.io

## When to Use PAW

- **Fuzzy search** — typo-tolerant matching, semantic search, near-duplicate detection
- **Format repair** — fix broken JSON, normalize dates, repair malformed inputs
- **Classification** — sentiment, urgency, categories defined in your own words
- **Extraction** — emails, names, dates from messy unstructured text
- **Log triage** — extract errors from verbose output, filter noise
- **Intent routing** — map user descriptions to the closest URL, menu item, or setting
- **Agent preprocessing** — parse tool calls, validate outputs, route tasks

## Install

```bash
pip install programasweights --extra-index-url https://pypi.programasweights.com/simple/
```

## Quickstart

```python
import programasweights as paw

# Use a pre-compiled function (downloads once, runs locally forever)
fn = paw.function("email-triage")
fn("Urgent: server is down!")  # "immediate"
fn("Newsletter: spring picnic")  # "wait"

# Compile your own from a description
program = paw.compile(
    "Fix malformed JSON: repair missing quotes and trailing commas"
)
fn = paw.function(program.id)
fn("{name: 'Alice',}")  # '{"name":"Alice"}'

# Or compile and load in one step
fn = paw.compile_and_load("Classify sentiment as positive or negative")
fn("I love this!")  # "positive"
```

If you want the smaller browser-compatible runtime explicitly, pass `compiler="paw-4b-gpt2"`. Otherwise, omit `compiler` and let the server default decide.

## Current Public Compilers

- **Standard** (`paw-4b-qwen3-0.6b`) — higher accuracy, 594 MB base + ~22 MB/program. This is the current server default.
- **Compact** (`paw-4b-gpt2`) — smaller (134 MB base + ~5 MB/program), runs in browser via WebAssembly.

Best practice:

- For quickstarts and reusable agent workflows, prefer `paw.compile(spec)` with no explicit compiler.
- If you need to target a specific runtime, pass `compiler="paw-4b-gpt2"` or another supported alias explicitly.
- If you need to inspect current server-supported compiler names at runtime, call `paw.list_compilers()`.

## Writing Good Specs

**The #1 practice: iterate with test cases.** Do not accept low performance on the first try. Build a test suite of input/output pairs, measure accuracy, then iteratively adjust wording and formatting until performance is good enough. Treat spec writing like software engineering: test, debug specific failures, fix the wording, retest.

A good spec has a description plus `Input: ... Output: ...` examples.

```python
fn = paw.compile_and_load("""
Classify user intent. Return ONLY one of: search, create, delete, other.

Input: Find the latest report
Output: search

Input: Make a new folder
Output: create

Input: Remove old backups
Output: delete
""")
```

**Spec-tuning tips:**

- **State output constraints explicitly**: "Return ONLY one of: X, Y, Z". Without this the model may produce free-form text.
- **Include examples from your actual data**: Examples outperform prose-only descriptions.
- **Debug failures before sweeping**: Look at specific failing examples and understand WHY before trying many variants.

## Constraints And Runtime Behavior

- Each PAW function is stateless: one text input, one text output. No conversation history.
- Spec + input + output share a ~2048 token context window. Inputs that exceed it will error.
- `max_tokens` defaults to `None`: generation runs until EOS or the context limit.
- Compile runs on the hosted PAW API. Inference should usually run locally through the SDK.
- **GPU acceleration** is enabled by default (`n_gpu_layers=-1`). Uses Metal on Mac, CUDA on Linux, and falls back to CPU automatically. If GPU causes issues, set `PAW_GPU_LAYERS=0` or pass `n_gpu_layers=0`.
- **First call** is usually ~1-5s because it loads the base model. Subsequent calls are typically ~0.05-0.5s depending on input length and GPU availability.
- **Base model files are shared** across programs on disk. Each Standard LoRA adapter is ~22 MB; each Compact LoRA adapter is ~5 MB.
- Cache root is `~/.cache/programasweights/`. Override with `PAW_CACHE_DIR`.
- After the first download, inference works offline.

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `RuntimeError: assets not ready` on download | Program is still generating after compile | The SDK polls automatically for up to 30s. If it still fails, retry shortly or recompile. |
| `httpx.HTTPStatusError: 422` on compile | Spec too short (<10 chars) or request validation failed | Adjust spec length or request shape. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [programasweights/programasweights-python](https://github.com/programasweights/programasweights-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
