---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Before acting, read the latest shared tensor4all agent rules from the
[`tensor4all-agent-rules`](https://github.com/tensor4all/tensor4all-agent-rules)
repository online. Start from:

- `https://github.com/tensor4all/tensor4all-agent-rules/blob/main/rules/index.md`

If internet access is unavailable or the remote cannot be resolved, use the
sibling checkout:

- `../tensor4all-agent-rules/rules/index.md`

Load only the common, Rust, performance, numerical, docs, or benchmark rule
files relevant to the task. If neither online access nor the sibling checkout
is available, continue from this repository's local rules and state the shared
rules were unavailable when creating a PR.

Then read the tenferro-specific rules:

- `REPOSITORY_RULES.md`

The sections below are tenferro-specific additions and overrides.

Before implementation work, review `REPOSITORY_RULES.md`.
Before creating a PR, review `REPOSITORY_RULES.md` again.
Before touching AD rules, oracle replay, or linearized boundary code, review `REPOSITORY_RULES.md` first.

## Current Implementation Status

The workspace contains active implementations alongside evolving APIs. Implementation work is allowed unless a task explicitly says otherwise.

## Repository Rule Source

Keep cross-repository implementation rules in `tensor4all-agent-rules`; do not
vendor a copy into this repository. Keep tenferro-specific durable rules in
`REPOSITORY_RULES.md`. This `AGENTS.md` file is the entry point and
tenferro-specific orientation; avoid duplicating the detailed performance,
layout, CPU kernel, slicing, cache, threading, and GPU backend rules here.

## Contribution Workflow Assets

Use repository-local contribution workflows when preparing external-facing
issues or bug-fix PRs:

- `ai/contribution-workflows/issue-intake.md` for bug reports, feature
  requests, design discussions, and documentation or article topic issues.
- `ai/contribution-workflows/bugfix-pr.md` for pull requests that fix existing
  intended behavior.
- `ai/contribution-workflows/repository-remediation.md` for batched,
  agent-assisted remediation of repository-rule violations across multiple
  related issues or findings.

Do not open a new-feature implementation PR before maintainers accept the
corresponding issue. If a proposed bug-fix PR needs a new public API, operation
family, backend, dependency, feature flag, architectural layer, or AD semantics
change, stop the PR path and use issue intake.

For batched repository-rule remediation work, follow
`ai/contribution-workflows/repository-remediation.md`. That workflow is a
deliberate exception to the normal one-bug-fix-PR path: collect all local fixes
and verification before opening a PR, keep coherent commits in a single PR, and
do not use squash merge.

Thin tool adapters live in `.agents/skills/`, `.claude/skills/`,
`.opencode/commands/`, and `.kimi/skills/`. Keep policy in `CONTRIBUTING.md` and
`REPOSITORY_RULES.md`; keep reusable workflow steps in
`ai/contribution-workflows/`.
See `ai/README.md` for the repository-local AI workflow layout.

### GPU Status

CUDA GPU support is implemented through the feature-gated CubeCL backend across
the concrete tensor, eager, and traced execution surfaces. Performance
optimization is still active work. The remaining CUDA limitations are specific:
`eig`, `full_piv_lu`, `full_piv_lu_solve`, `dynamic_update_slice`, `I64`
numeric/linalg gaps, and selected complex analytic or ordering operations.
HIP/ROCm remains stubbed. Outside explicit GPU implementation tasks, check
`docs/guides/devices-and-gpu.md` and the current CUDA/CubeCL backend tests
before assuming a specific op/dtype/backend combination is supported.

### Documentation Requirements

Every public type, trait, and function **must** include minimal but sufficient usage examples in its doc comments (`/// # Examples`). The examples should help a human quickly understand how to use the API. Doc examples must compile and run as doctests; do not use `ignore` or `no_run`. Crate-level docs (`//!`) should include typical end-to-end usage examples.

## Project Overview

**tenferro-rs** is a general-purpose tensor computation library in Rust (`tenferro-*` crates). It provides:
- Dense tensor types with CPU/GPU placement metadata
- Graph-based traced execution via `TracedTensor` + `GraphExecutor`
- Standard extension crates for operation families such as einsum, linalg, and FFT
- Automatic differentiation (VJP/JVP/HVP) for the standard dense numeric path
- Single execution IR (`ExecOp`) plus a pass pipeline for backend dispatch

**strided-rs** (separate workspace) is an external foundation dependency providing:
- `strided-traits`: `ScalarBase`, `ElementOp` traits
- `strided-view`: Dynamic-rank strided views (`StridedView`/`StridedViewMut`)
- `strided-kernel`: Cache-optimized map/reduce/broadcast kernels

tenferro-rs depends on strided-rs but does not absorb it. strided-rs has no BLAS dependency and can be used standalone.

### Design Documents

See [`docs/design/`](docs/design/) for architecture and design documents.

### Work Logs And Review Intent


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tensor4all/tenferro-rs](https://github.com/tensor4all/tenferro-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
