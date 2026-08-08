---
trigger: always_on
description: Guidelines for Codex, opencode, Claude Code, and other coding agents working in
---

# AGENTS.md — NPU Agent Workspace System Prompt

Guidelines for Codex, opencode, Claude Code, and other coding agents working in
this repository.

This repository contains reusable AI coding workflows for NPU large-model serving
optimization. The first supported landing target is xLLM, but the evidence
model should stay portable to other OpenAI-compatible NPU serving frameworks
when their adapters and runbooks are added.

## 1. Repository Purpose

- Preserve repeatable NPU optimization workflows as skills, schemas, scripts,
  prompts, and model history.
- Keep benchmark, profiling, accuracy, capacity, and review evidence explicit.
- Separate generic workflow rules from model-specific or framework-specific
  lessons.
- Promote durable lessons into `reference/pr_history/`, `reference/`,
  `skills/*/references/`, or run-root `humanize/` ledgers.

## 2. Core Engineering Constraints

1. **Think Before Editing**
   - State assumptions when the task is ambiguous.
   - If multiple interpretations change the implementation, ask before editing.
   - If a simpler approach exists, say so before choosing a heavier path.
   - Prefer the existing skill structure, artifact schema, and naming conventions.
   - For trivial documentation fixes, make the small obvious edit and verify it.

2. **Simplicity First**
   - Implement only what the user asked for and what the validation goal requires.
   - Do not add abstractions, configuration, compatibility layers, or speculative
     features for a single-use need.
   - If a change starts to sprawl, shrink it back to the smallest verifiable diff.

3. **Evidence Before Patch**
   - Performance optimization requires a warmed-up baseline and profiling evidence
     before code changes.
   - Accuracy fixes require a stable reproducer before broader evaluation.
   - Profiling captures explain bottlenecks; they are not formal before/after
     performance results.
   - Do not claim a gain without raw artifacts, metrics, and the exact workload.
   - For end-to-end goals, rank L0 architecture, L1 pipeline, and L2 operator
     candidates before entering L3 detail. Select by expected end-to-end gain,
     not by ease of implementation.

4. **Keep Changes Surgical**
   - Touch only files needed for the request.
   - Do not rewrite skill bodies into long essays. Keep `SKILL.md` procedural and
     move detailed material into `references/` when needed.
   - Do not delete failed attempts or historical lessons; convert them into
     reusable notes.
   - Do not add local paths, private host names, internal IPs, private datasets, or
     secrets to committed files.

5. **Fair Comparisons Only**
   - Compare frameworks under the same model, tokenizer, dtype, hardware, workload,
     sampling parameters, and SLA. Tune each framework independently.

6. **Profiling Is Diagnostic**
   - Profiling captures explain bottlenecks but do not replace non-profiling
     before/after performance runs.

7. **Review-Gated Evidence Loop**
   - Use Research → Learn → Code → Review → Validate → Record.
   - This is inspired by PolyARCH/humanize's RLCR review discipline, but this
     repository does not implement Humanize RLCR itself.

8. **Validate and Record**
   - Run repository tests after changing schemas, scripts, or skill structure.
   - For documentation-only edits, at least run markdown-sensitive hygiene checks
     when available.
   - Update README / README_zh / AGENTS.md together when changing public workflow
     concepts.
   - End every optimization or bug-fix loop by recording reusable lessons in a
     ledger, reference, or model PR history.

9. **Use The Unified Lifecycle For New Work**
   - Define new experiments from `reference/io_specs/experiment.example.yaml`.
   - Run `scripts/xllm_flow.py preflight` before execution.
   - Create and resume run roots through `run create` and `checkpoint`.
   - Record each candidate with `attempt add`; do not rerun a completed
     fingerprint without an explicit repeat index.
   - Use `run validate` before finalization and keep generated ledgers and
     checksums with the run root.
   - Finish with `run finalize`; archive only after evidence and retention
     decisions are recorded.

## 3. Task → Skill Routing

| Task | Start With |
|---|---|
| Query model optimization history, prior risks, or PR lessons | `skills/model-pr-optimization-history/SKILL.md` |
| Create, resume, validate, finalize, or archive an experiment run | `skills/xllm-experiment-lifecycle/SKILL.md` |
| End-to-end optimization goal | `skills/xllm-npu-sota-loop/SKILL.md` |
| Build, compile, build gate, NPU gate, or multi-candidate build reuse | `skills/xllm-npu-build-gate/SKILL.md` |
| End-to-end eval (service + perf + accuracy + report) | `skills/xllm-npu-eval-runner/SKILL.md` |
| Batch perf eval for multiple models with different TP | `skills/xllm-npu-batch-perf/SKILL.md` |
| Launch, stop, or health-check xLLM service | `skills/xllm-npu-server-manager/SKILL.md` |
| Run evalscope performance test only | `skills/xllm-npu-perf-runner/SKILL.md` |
| Run evalscope accuracy test only | `skills/xllm-npu-accuracy-runner/SKILL.md` |
| Generate report from existing artifacts | `skills/xllm-npu-report-writer/SKILL.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xLLM-AI/xllm-workflow](https://github.com/xLLM-AI/xllm-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
