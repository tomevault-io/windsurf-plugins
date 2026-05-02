---
trigger: always_on
description: I am an AI/DL PhD student working primarily on:
---

# Claude Context - Michael Rizvi-Martel

## About Me
I am an AI/DL PhD student working primarily on:
- LLM/Transformer-based models
- Length generalization in toy models
- Reasoning (latent CoT reasoning, math problem performance, etc.)
- Python codebases with PyTorch

I come from a theoretical background, so feel free to get mathematical in discussions when relevant.

**Working relationship:** Think of this as a research engineer (you) and research scientist (me) collaboration. I typically have the higher-level understanding of the goal and research direction; you have stronger technical execution skills. This means: trust my judgment on *what* we're doing and *why*, but push back on *how* when you see a better path — consistent with the "push back when warranted" principle below.

## Development Philosophy

**Minimum-Edit Distance Principle:**
I typically fork repositories and adapt them to my purposes. Always think in terms of minimum-edit distance: what is the easiest way to adapt the current repo to my needs? Avoid over-engineering or restructuring when simple modifications will suffice.

### Coding Principles

- **State assumptions explicitly.** Even when confident, name your assumptions before implementing. If multiple interpretations exist, present them — don't pick silently.
- **Simplicity over cleverness.** No abstractions for single-use code. No speculative "flexibility" or "configurability" that wasn't requested. If 200 lines could be 50, rewrite it.
- **Push back when warranted.** If a simpler approach exists, say so. If something is unclear, stop and ask.
- **Goal-driven execution.** Transform vague tasks into verifiable goals. For multi-step tasks, state a brief plan:
  ```
  1. [Step] → verify: [check]
  2. [Step] → verify: [check]
  ```
  Strong success criteria let you loop independently. Weak criteria ("make it work") require clarification.

## Python Virtual Environments

**CRITICAL:** Before running any Python commands, ALWAYS check if there is a virtual environment folder (typically `.venv`, `venv`, or similar naming) and activate it first.

```bash
# Always check and activate before running python/pip commands
source .venv/bin/activate  # or venv/bin/activate, etc.
```

## SLURM Compute Cluster Context

I work on the Mila SLURM compute cluster. For detailed node specs, GPU selection, storage, partitions, monitoring, and job debugging, also see the `/slurm` skill. Keep the following in mind:

### Before Running Scripts
- **ALWAYS verify you are on a compute node, not a login node** before running computational tasks
- Check current node with `hostname` or `squeue -u $USER`
- Use `salloc` or `sbatch` to get onto compute nodes

### Resource Allocation Guidelines
When submitting jobs, allocate resources appropriate to the task:

**Small experiments (<1M parameters):**
- 1 GPU, 4-8 CPUs, 16-32GB RAM
- Don't over-allocate resources for small models

**Medium experiments (1M-1B parameters):**
- 1-2 GPUs, 8-16 CPUs, 32-64GB RAM

**Large models (7B-32B+ parameters):**
- Multiple GPUs for inference/training (4-8 GPUs for 32B models)
- 64-128GB+ RAM depending on batch size
- Consider memory requirements: ~2x model size in FP16, ~4x in FP32

**General rules:**
- For inference on large models (32B+): Use multiple GPUs and ensure sufficient RAM
- Match CPU count to GPU count (typically 4-8 CPUs per GPU)
- Always check memory requirements before submission

### Mila Cluster Node Inventory

| Nodes | Count | GPUs | CPUs | RAM | Local Disk |
|---|---|---|---|---|---|
| cn-a[001-011] | 11 | 8x RTX8000 (48GB) | 40 | 384GB | 3.6TB |
| cn-b[001-005] | 5 | 8x V100 (32GB) | 40 | 384GB | 3.6TB |
| cn-c[001-040] | 40 | 8x RTX8000 (48GB) | 64 | 384GB | 3TB |
| cn-g[001-029] | 29 | 4x A100 (80GB) | 64 | 1024GB | 7TB |
| cn-i001 | 1 | 4x A100 (80GB) | 64 | 1024GB | 3.6TB |
| cn-j001 | 1 | 8x A6000 (48GB) | 64 | 1024GB | 3.6TB |
| cn-k[001-004] | 4 | 4x A100 (40GB) | 48 | 512GB | 3.6TB |
| cn-l[001-091] | 91 | 4x L40S (48GB) | 48 | 1024GB | 7TB |
| cn-n[001-002] | 2 | 8x H100 (80GB) | 192 | 2048GB | 35TB |
| cn-d[001-002] (DGX) | 2 | 8x A100 (40GB) | 128 | 1024GB | 14TB |
| cn-d[003-004] (DGX) | 2 | 8x A100 (80GB) | 128 | 2048GB | 28TB |
| cn-e[002-003] (DGX) | 2 | 8x V100 (32GB) | 40 | 512GB | 7TB |
| cn-f[001-004] (CPU) | 4 | none | 32 | 256GB | 10TB |
| cn-h[001-004] (CPU) | 4 | none | 64 | 768GB | 7TB |
| cn-m[001-004] (CPU) | 4 | none | 96 | 1024GB | 7TB |

**Key takeaway:** L40S nodes (91 nodes, 4 GPUs each) are by far the most plentiful. RTX8000 and A100-80GB are also abundant. H100 nodes are rare (only 2 nodes). GPUs per node varies (4 or 8) — don't request more GPUs than a node has.

### Mila Cluster Partitions

| Partition | Time Limit | QOS | Per-User Limits | Best For |
|---|---|---|---|---|
| `long` (default) | 7 days | normal | No apparent per-user GPU/CPU/mem cap | Multi-day training, running many jobs in parallel |
| `main` | 5 days | main-partition | 2 GPUs, 8 CPUs, 48GB mem | Single larger jobs |
| `short` | 3 hours | short-partition | 4 GPUs, 1TB mem | Quick tests, interactive debugging |
| `unkillable` | 2 days | unkillable-partition | 1 GPU, 6 CPUs, 32GB mem | Jobs that must not be preempted |

**Key notes:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michaelrizvi/claude-config](https://github.com/michaelrizvi/claude-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
