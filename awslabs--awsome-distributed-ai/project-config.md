---
trigger: always_on
description: > These instructions apply to **all** AI-assisted contributions to
---

# Agent Instructions for awsome-distributed-ai

> These instructions apply to **all** AI-assisted contributions to
> `awslabs/awsome-distributed-ai`. They complement
> [CONTRIBUTING.md](./CONTRIBUTING.md), which remains the authority on
> contribution format and process.

## 1. Contribution Policy

### Duplicate-work checks

Before proposing a PR, run these checks:

```bash
gh issue view <issue_number> --repo awslabs/awsome-distributed-ai --comments
gh pr list --repo awslabs/awsome-distributed-ai --state open --search "<issue_number> in:body"
gh pr list --repo awslabs/awsome-distributed-ai --state open --search "<short area keywords>"
```

- If an open PR already addresses the same fix, do not open another.
- If your approach is materially different, explain the difference in the issue.

### No low-value busywork PRs

Judge a change by its impact, not its diff size. An atomic fix is welcome
when it corrects a concrete functional, correctness, or user-facing defect
(a broken command, a wrong path, a bug in a script) with proportionate
verification. Do not open PRs for edits with no concrete impact (pure style
preference, cosmetic rewording), do not bundle unrelated cleanup, and do
not pad a focused fix to make it look substantive.

### Accountability

- A human submitter must understand and defend the change end-to-end, review
  every changed line, and run the relevant verification.
- This repository ships **runnable reference material, not a library**: the
  test for a change is that the example actually works on the target
  hardware. CONTRIBUTING.md already requires test cases to be verified at
  their stated scale ("if you say 256 A100, test on that scale").
- PR descriptions for AI-assisted work **must** include:
  - Why this is not duplicating an existing PR.
  - How the change was verified, proportionate to the change:
    - **New or functionally changed example**: an end-to-end run on
          the target hardware — cluster type, instance types, and scale
          used, with relevant command output or logs under
          `## Test Results`. Running this e2e verification is the PR
          author's responsibility.
    - **Small functional fix**: verification proportionate to the
          change (a build, a targeted test, or a reviewable demonstration
          of the fix).
    - **No functional impact**: write `docs-only` under
          `## Test Results`.

### Fail-closed behavior

If the work is duplicate or trivial busywork, or you cannot provide the
verification required above, **do not proceed**. Return a short
explanation of what is missing.

## 2. Repository Layout — where things go

| Directory | Contents |
| --- | --- |
| `architectures/` | Cluster-level reference architectures, one per orchestrator (ParallelCluster, PCS, HyperPod Slurm/EKS, EKS) plus shared building blocks (`common/`, `vpc_network/`, `ldap_server/`, `accounting-database/`) |
| `examples/training/` | Framework-centric training examples (FSDP, Megatron-LM, NeMo, verl, …) |
| `examples/use-cases/` | Model- or application-specific examples (fine-tunes, domain pipelines, world models, VLA models, …) |
| `examples/inference/` | Serving examples, organized by engine (vLLM, SGLang, NVIDIA Dynamo, …) |
| `micro-benchmarks/` | Low-level performance benchmarks (NCCL tests, expert-parallelism kernels, NVSHMEM, …) |
| `validation/` | Environment and cluster health validation (healthchecks, env screening) |
| `observability/` | Monitoring stacks, metrics exporters, profiling (Prometheus/Grafana, Nsight) |
| `ami/` | Machine image build assets (Packer/Ansible) |
| `docs/` | Cross-cutting prose documentation (e.g. the [EFA cheatsheet](./docs/efa-cheatsheet.md)) |

> **Exception**: two `LifecycleScripts` trees are deliberately retained at
> their legacy numbered paths under `1.architectures/` for SageMaker
> HyperPod console compatibility (see
> [`1.architectures/README.md`](./1.architectures/README.md)). Do not
> move, rename, or delete them as cleanup.

Placement rules for new content:

- **Extend before create** (applies repo-wide — examples,
  micro-benchmarks, and architectures alike). Before adding a new
  directory, check whether an existing one already covers the same
  framework, engine, or benchmark. Prefer updating it — a new variant
  subdirectory, model recipe, or README section — over adding a parallel
  sibling. A new directory is justified only when its dependencies or
  execution model differ materially from the closest existing one; name
  that closest sibling in the PR and explain why extending it is not
  enough. Existing sibling directories are not presumed duplicates — do
  not consolidate them retroactively under this rule. Unbounded
  near-duplicate directories are a maintenance liability.
- A **training framework** example goes to `examples/training/<framework>/`.
- A **specific model or application** (even if it trains) goes to
  `examples/use-cases/<name>/`.
- Anything whose purpose is **serving** goes to
  `examples/inference/<engine>/<name>/`.
- Kernel- or transport-level performance measurement goes to
  `micro-benchmarks/`.
- Every example is **self-contained**: its own README (prerequisites,
  copy-pasteable run instructions, known issues), pinned dependency

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awslabs/awsome-distributed-ai](https://github.com/awslabs/awsome-distributed-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
