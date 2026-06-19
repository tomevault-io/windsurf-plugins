---
trigger: always_on
description: Create, revise, audit, or launch bounded autonomous ML/software research loops with protected baselines, tiered evals, lineage, metric investigations, leakage pre-flight checks, and biology no-regression checks. Agent-agnostic — produces prompts usable by any coding agent (Claude Code, ChatGPT / Codex, Cursor, Aider, or custom agent harnesses).
---


# Autoresearch-Bio Skill

This is a **bio-first, domain-general, agent-agnostic** skill for creating and supervising bounded, hypothesis-driven autonomous research loops. It is strongest for biological ML, but the core discipline also applies to general ML, software engineering, agent development, benchmarking, infrastructure, and developer-tooling experiments.

The skill's primary artifact is a paste-ready `autoresearch.md` **prompt for a coding agent** — any coding agent. Tested invocation paths include Claude Code, ChatGPT (Codex / GPT agents), Cursor, Aider, and bespoke agent harnesses built on the Anthropic, OpenAI, or open-source model APIs. The skill itself is written in plain Markdown and contains no vendor-specific tool calls; you can also use it directly as a human-authored prompt template without an agent at all.

The skill can also produce amendments, decision memos, metric investigation prompts, or reusable skill updates.

**Roles.** Producing an `autoresearch.md` and running it are two jobs. A strong reasoning chat model is usually the better **planner** (reads context, designs the families, tiers, metrics, baselines, stop conditions) and a coding agent is the better **executor**. When this skill is invoked in a chat model to draft a prompt, it is the planner; the file it emits is run by the executor. The user can supply the whole design, ask the planner to design it from a bare problem, or do a hybrid. `references/planner_workflow.md` covers the split and how to prompt the planner; the family-set switches live in `core_protocol.md §5`. The discipline invariants do not change with the role.

The invariant is the same in every domain: protect the model or system of record, register baselines before search, use tiered gates, track lineage of experiments, document every result, avoid metric loopholes, and stop cleanly when stop conditions fire.

---

## Golden Path

When invoked:

1. Identify the requested artifact type:
   - new `autoresearch.md`;
   - session amendment;
   - continue/amend/audit/close decision memo;
   - metric investigation prompt;
   - reusable skill update.
2. Identify the domain:
   - biology or scientific ML;
   - general ML;
   - software/developer tooling;
   - infrastructure/performance;
   - agent, benchmark, or evaluation loop;
   - other bounded experiment loop.
3. Identify or request only the missing essentials:
   - model or system of record;
   - datasets, benchmarks, CI suites, simulator scenarios, traffic slices, or split roles;
   - primary, secondary, protected, and catastrophic-fail metrics;
   - observed failure modes;
   - compute or engineering budget;
   - locked files, identity constraints, and evaluator constraints;
   - autonomy mode.
4. If enough information exists, produce a self-contained artifact. Do not give vague advice when a paste-ready prompt is possible.
5. If essential information is missing, ask at most five prioritized questions. If the user asks for a best-effort draft, proceed with explicit assumptions and mark unknown thresholds as `TO_FILL_BEFORE_LAUNCH`.
6. Always enforce the core invariants:
   - protected model or system of record;
   - Step 0 baselines before architecture/search/development loops;
   - tiered gates;
   - exact keep/discard labels;
   - no Tier 1 or Tier 2 rebasing;
   - lineage tracking via parent_experiment_ids and branch_type;
   - documented stop conditions;
   - domain-appropriate safety boundaries.

---

## When To Use This Skill

Use this skill when the user asks for help with a repository, experiment log, trained model, software system, agent, benchmark, or research loop and wants to:

- draft an `autoresearch.md` prompt;
- set up an autonomous architecture, mechanism, benchmark, or implementation search;
- extend a trained biological ML model while protecting an existing baseline;
- improve a non-bio model or software system while protecting correctness, safety, latency, memory, slice, or robustness behavior;
- convert experimental lessons into a reusable research protocol;
- decide whether to continue, amend, audit, or close an autonomous loop;
- investigate metrics before reopening architecture or implementation search;
- add Debate Council autonomy to a long-running experiment loop;
- add lineage tracking to a flat experiment log that is hiding combinatorial search structure.

Typical trigger phrases include:

- "Help me draft an autoresearch prompt for this repo."
- "I want Codex to run experiments on this biology model."
- "Use this skill for my non-bio dev project."
- "Design a protected-baseline research loop for this benchmark."
- "What should the agent do next after these failed experiments?"
- "Should I reopen the architecture search?"
- "Design a metric investigation for my perturbation model."
- "Turn these single-cell experiment failures into a reusable protocol."
- "Make an autoresearch loop for improving this code-generation agent without regressing safety or latency."


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yashraj59/autoresearch-bio](https://github.com/yashraj59/autoresearch-bio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
