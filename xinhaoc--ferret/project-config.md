---
trigger: always_on
description: Autonomous CUDA kernel optimization agent. Takes a task.yaml spec, launches a motus ReActAgent on B200 GPUs, writes/edits/benchmarks kernels, tracks versions with git.
---

# Ferret — CUDA Kernel Optimization Agent

Autonomous CUDA kernel optimization agent. Takes a task.yaml spec, launches a motus ReActAgent on B200 GPUs, writes/edits/benchmarks kernels, tracks versions with git.

## Remote setup

- Machine: `catalyst-fleet1` (shared cluster, B200 GPUs)
- Ferret repo: `~/repos/ferret`
- Python: check `scripts/run.sh` for PYTHON path (was miniconda, may change)
- `scripts/run.sh` handles workspace init + launch

## Launching a run

```bash
# Fresh start (wipes workspace):
~/repos/ferret/scripts/run.sh tasks/<task>.yaml --max-iterations 60

# Resume (keeps workspace + git history) — DEFAULT CHOICE for continuing work:
~/repos/ferret/scripts/run.sh tasks/<task>.yaml --max-iterations 60 --keep-workspace
```

**Always default to `--keep-workspace` unless explicitly starting a new task.** Fresh start throws away hours of agent work. Every time you give a fresh command when it should be resume, hours of GPU time and API tokens are wasted.

Before launching: `cd ~/repos/ferret && git pull --ff-only` to get latest code.

## Workspace lifecycle

- `workspace/` has its own `.git` (separate from parent ferret repo)
- Agent commits `v###` tags for improvements, `a###` for failed attempts
- `workspace/` is in `.gitignore`
- **Wipe correctly**: `rm -rf workspace && mkdir workspace` (NOT `rm -rf workspace/*` — misses `.git` dotfile, agent then reads old git history from parent repo)
- Save valuable workspaces to `legacy/<task-name>/` before wiping
- Save best kernels to `examples/` so they persist across workspace wipes

## Creating a task.yaml

### Checklist

1. **Task yaml** in `tasks/` — see `tasks/template.yaml`
2. **Baseline script** in `baselines/` — must be runnable, agent calls it for KERNEL_RESULT_REFERENCE
3. **References** — proven working kernels first (agent reads top-down), then library source
4. **Validate**: `python3 task_spec.py tasks/<task>.yaml`
5. **Validate refs**: `python3 scripts/check_resource_refs.py`
6. **Commit + push immediately** — never wait for user to discover files are missing

### baseline.source

A label (e.g. "cuBLAS", "FA2 unabsorbed"), NOT a filesystem path. `main.py` validates `references[]` paths, not `baseline.source`. The agent reads `baseline.source` to know WHAT to measure, then runs the baseline script and uses its output for KERNEL_RESULT_REFERENCE.

### references

Filesystem paths the agent reads during REPRODUCE. Put the most relevant example first — agent reads top-down. Always include:
- Best prior kernel for this task family
- Relevant tcgen05/PTX examples from `examples/tcgen05-gemm/`

### constraints vs hints

- **constraints**: framework rules only. "cta_group::1 only", "single stream", "no CUDA graphs". Injected every iteration. Do NOT put optimization suggestions here — that constrains the agent's exploration.
- **hints**: first-turn only. Keep factual. Do NOT add opinions about what the agent should or shouldn't try.

## Critical lessons (from painful experience)

### FLOPS formula consistency

The #1 recurring bug. The kernel's benchmark and the baseline script MUST use the same FLOPS formula. If one uses `2 * B * H * S * S * D` (standard multiply-accumulate) and the other uses `B * H * S * S * D` (1x), all ratios are 2x inflated. Check BOTH formulas before trusting any ratio.

The kernel's formula is in `kernel.cu` (search for `double fl=`). The baseline's is in `baselines/*/baseline*.py` (search for `flops =`). They must match.

### Agent hardcodes reference numbers

The agent often measures the baseline once, hardcodes KERNEL_RESULT_REFERENCE in kernel.cu's benchmark, then never re-measures. If the FLOPS formula changes (yours or the agent's), the hardcoded reference becomes stale. The prompt says "run baselines/ script every time you tag" but agents ignore this.

Verify: after each run, check that KERNEL_RESULT_REFERENCE in the commit body matches what the baseline script actually produces on the same GPU.

### Agent picks the wrong baseline when multiple are printed

If the baseline script prints multiple references (e.g. trtllm + FA2), the agent picks whichever is most favorable. Fix: print only the target baseline. Remove alternatives or clearly label which one to use for scoring.

### MLA prefill: absorbed vs unabsorbed

- **Absorbed** (D_QK=576, D_V=512): skips kv_b_proj decompression. 3x more compute per head. Wins at short S (S≤1024) where memory dominates. Loses at long S where compute dominates.
- **Unabsorbed** (D_QK=192, D_V=128): standard MHA after decompression. What vLLM/SGLang deploy for prefill. Less compute, faster at long S.
- `trtllm_batch_decode_with_kv_cache_mla` is the ONLY trtllm API for absorbed-form MLA (handles asymmetric QK/V dims). Despite "decode" in the name, it works for any Q_LEN.
- `trtllm_batch_context_with_kv_cache` does NOT support MLA (requires headDimQk == headDimV).
- For unabsorbed prefill baseline: `BatchPrefillWithRaggedKVCacheWrapper` (FA2 JIT) or `single_prefill_with_kv_cache` (CUTLASS SM100a, faster on B200).

### FlashInfer has multiple FA2 implementations

On B200:
- `BatchPrefillWithRaggedKVCacheWrapper` → FA2 JIT kernel (what SGLang uses in production)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xinhaoc/ferret](https://github.com/xinhaoc/ferret) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
