---
trigger: always_on
description: torch-xpu-ops provides XPU (Intel GPU) operator implementations for PyTorch ATen.
---

# Coding Agent Guide for torch-xpu-ops

## Project Overview

torch-xpu-ops provides XPU (Intel GPU) operator implementations for PyTorch ATen.
It is built as a submodule of PyTorch (`third_party/torch-xpu-ops`), not standalone.

Before reviewing, implementing, or debugging any operator, you **MUST**
read the corresponding upstream PyTorch code to understand the expected
semantics and some patterns:

- `src/ATen/native/xpu/<Op>.cpp` → upstream: `aten/src/ATen/native/<Op>.cpp`
- `src/ATen/native/xpu/sycl/<Op>Kernels.cpp` → upstream CUDA equivalent: `aten/src/ATen/native/cuda/<Op>.cu`
- Dispatch registration → upstream: `aten/src/ATen/native/native_functions.yaml`

Use `gh` CLI, WebFetch, or sub-agents to fetch files from the `pytorch/pytorch`
repository. Do not infer upstream behavior from memory — verify from source.
This applies to code review, new implementations, bug fixes, and understanding
existing logic.

## Working Principles

### Think Before Coding

Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### Simplicity First

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.

### Surgical Changes

Touch only what you must. Clean up only your own mess.

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### Goal-Driven Execution

Define success criteria. Loop until verified.

Transform tasks into verifiable goals:
- "Add validation" -> "Write tests for invalid inputs, then make them pass"
- "Fix the bug" -> "Write a test that reproduces it, then make it pass"
- "Refactor X" -> "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] -> verify: [check]
2. [Step] -> verify: [check]
3. [Step] -> verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## Repository Structure

```
src/ATen/native/xpu/          # Dispatch registration + thin wrappers (no SYCL)
src/ATen/native/xpu/sycl/     # SYCL kernel implementations (.cpp/.h)
src/ATen/native/nested/xpu/   # Nested tensor XPU implementations
src/comm/                      # Shared utility headers
src/xccl/                      # XCCL communication backend
test/xpu/                      # XPU device tests (pytest-based)
test/regressions/              # Regression tests
test/sycl/                     # SYCL C++ unit tests (CMake)
tools/linter/                  # Lint adapter scripts
```

## Scratch Space

Use `agent_space_xpu/` (git-ignored, at repo root) for temporary scripts, scratch files, and throwaway experiments. Do not commit files from this directory.

## PR Review

When asked to review a PR, always use the /pr-review skill.

## Environment

If any tool you're trying to use (pip, python, spin, etc) is missing, check for
a `.venv` directory in the project root or its parent directory. If found,
activate it and retry. If no `.venv` is found, stop and ask the user if an
environment is needed. Do NOT try to find alternatives or install these tools.

## CI Docker Images

The `.ci/docker/` directory is content-hashed to determine whether Docker images
need rebuilding. Any file change inside `.ci/docker/` (including the README)
changes the hash and triggers a full Docker image rebuild. Do not make changes
in this directory unless you intend to rebuild Docker images. When Docker builds
are broken (e.g., due to an upstream Ubuntu outage), avoid touching this
directory so you don't force a rebuild against the broken state.

## Commit Messages

Don't commit unless the user explicitly asks you to.

When writing a commit message, don't make a bullet list of the individual
changes. Instead, if the PR is large, explain the order to review changes
(e.g., the logical progression), or if it's short just omit the bullet list
entirely.

The commit message should be clear, informative, and have a Test Plan section
that describes how you tested the change. If you are fixing a bug, the commit
message must explain the root cause of the bug and how the fix works.
If there were multiple potential paths you could have taken, please call them
out succinctly and justify the one you took.

When describing the testing strategy in a commit message, include the literal
commands that were run in fenced Markdown code blocks.

Disclose that the PR was authored with an AI assistant.

When the user asks you to amend a commit, check whether the commit message

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intel/torch-xpu-ops](https://github.com/intel/torch-xpu-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
