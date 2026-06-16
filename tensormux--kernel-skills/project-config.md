---
trigger: always_on
description: kernel-skills is an open source repository of extremely high quality skill files for AI coding agents.
---

# CLAUDE.md

# kernel-skills
## Repository mission

kernel-skills is an open source repository of extremely high quality skill files for AI coding agents.

The purpose of this repository is to help agents write better compute kernels across CUDA, Triton, quantization-oriented kernels, and core optimization patterns. This repository is not a runtime, framework, package manager, or MCP server in its first iteration. It is a curated and disciplined library of `SKILL.md` files that agents and humans can directly use.

The repository must be public, contribution-friendly, easy to browse, and immediately useful without any tooling beyond reading and copying Markdown files.

The central principle is simple:

**Every skill in this repository must materially improve the quality, correctness, and performance-awareness of kernel code produced by an AI agent.**

---

## What this repository is

This repository is:

- a library of structured `SKILL.md` files
- focused on kernel engineering for AI and high performance numerics
- designed for use with coding agents like Claude Code, ChatGPT, Cursor, Gemini CLI, Codex, and similar tools
- optimized for practical reuse and public OSS contribution
- strict about quality, technical accuracy, and clarity

This repository is not:

- a kernel runtime
- a benchmark harness platform
- an MCP integration project
- a code execution framework
- a dumping ground for vague prompts
- a tutorial-only repo
- a place for low-quality or repetitive skills

---

## Product scope for v1

Focus only on skills.

Do not build packaging systems, CLI tools, MCP servers, benchmark runners, or automation infrastructure in the first iteration unless absolutely required for basic repo hygiene.

The first version should prove one thing:

**well-authored skill files can help AI agents produce significantly better kernel code**

---

## Core outcome we want

A user should be able to open any skill in this repository, paste it into their agent workflow, and get a much better result than they would get from a generic prompt.

A "much better result" means the agent is more likely to:

- gather the right constraints before coding
- choose the right kernel strategy
- avoid common correctness bugs
- reason about memory hierarchy properly
- reason about tiling and launch decisions properly
- avoid fake or hand-wavy performance claims
- explain tradeoffs with technical precision
- generate code that is more realistic, performant, and maintainable

---

## Non-negotiable quality standard

This repository must aim for expert-grade skill files.

Every skill must push the agent toward:

- correctness first
- performance-aware design
- realistic optimization reasoning
- explicit tradeoff analysis
- architecture-sensitive decisions
- honest uncertainty when information is missing

Every skill must avoid:

- empty boilerplate advice
- vague "optimize this" style language
- cargo-cult GPU optimization
- unsupported claims about speedups
- assuming custom kernels are always the right answer
- ignoring boundary conditions, alignment, layouts, or numerical effects

Important:
The repository should aggressively bias toward high performance kernels, but must never fake authority. A good skill does not force a custom kernel when cuBLAS, cuDNN, CUTLASS, vendor kernels, or higher-level libraries are the better answer. The skill should explicitly say so when appropriate.

---

## Engineering philosophy

This repository should embody the mindset of a serious kernel engineer.

That means the skills should lead the agent to reason about:

- algorithmic structure before micro-optimization
- memory access before arithmetic intensity claims
- correctness before speed
- measurement before bragging
- architecture constraints before generic advice
- maintainability and fallback paths when appropriate

The best skill files are not merely instructive.
They are decision frameworks.

---

## Scope of kernel types to cover

The first repository version should focus on compute kernels relevant to AI workloads and performance engineering.

### In scope
- CUDA kernels
- Triton kernels
- reduction kernels
- GEMM-related kernels
- softmax kernels
- layernorm kernels
- attention-related kernels
- fused elementwise kernels
- quantized kernels such as int8 and fp8-oriented workflows
- optimization patterns such as tiling, memory coalescing, shared memory usage, warp divergence avoidance, and launch configuration design
- debugging and correctness-focused kernel skills
- portability planning such as CUDA to Triton or CUDA to HIP planning

### Out of scope for the first version
- Linux kernel development
- device driver internals
- graphics shader pipelines as a primary focus
- full compiler toolchain design
- execution tooling
- benchmarking infrastructure as product surface
- packaging into MCP or other distribution frameworks

---

## Repository structure to create

Create the repository with the following structure:

```text
kernel-skills/
├─ README.md
├─ LICENSE
├─ CONTRIBUTING.md
├─ CODE_OF_CONDUCT.md
├─ ROADMAP.md
├─ CLAUDE.md
├─ .gitignore
├─ skills/
│  ├─ cuda/
│  │  ├─ write-cuda-gemm-kernel/
│  │  │  └─ SKILL.md
│  │  ├─ write-cuda-reduction-kernel/
│  │  │  └─ SKILL.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tensormux/kernel-skills](https://github.com/tensormux/kernel-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
