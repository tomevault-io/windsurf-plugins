---
trigger: always_on
description: Primary artifact: `ROSLYN_AGENTIC_CODING_RESEARCH_PROPOSAL.md`
---

# Roslyn Agentic Coding Research

Primary artifact: `ROSLYN_AGENTIC_CODING_RESEARCH_PROPOSAL.md`  
This file (`AGENTS.md`) defines how agents should execute the project at maximum practical speed while preserving scientific rigor.

## Project State

Current mode: planning and design, rapidly transitioning to implementation once dependency prerequisites are satisfied.  
Primary objective: prove or disprove that Roslyn-native agent tooling materially outperforms text-first editing workflows for C#/.NET.
Priority order: tooling quality -> benchmark rigor -> meta-learning quality -> industrial research report.

## Distilled External Lessons (Meta-Study Inputs)

These principles are distilled from recent work/posts/talks by:

- Andrej Karpathy
- Erik Meijer (`@headinthebox`)
- Jeffrey Emanuel (`@doodlestein`, `Dicklesworthstone`)
- Peter Steinberger (`@steipete`)
- Paul Gauthier (Aider)

Synthesis to apply here:

- Keep autonomy high but controllable: human sets intent/constraints; agents execute aggressively inside clear guardrails.
- Treat planning as first-class code: do not "wing it"; produce explicit operation plans before large edits.
- Use compiler/semantic systems as truth sources, not optional helpers.
- Measure trajectories and outcomes, not just individual edits.
- Optimize for iteration velocity via task decomposition and parallelism, not by lowering quality bars.

## Applied Heuristics by Source

Karpathy-inspired:

- Use an autonomy slider: increase agent autonomy only when objective, constraints, and guardrails are explicit.
- Keep the human in high-level supervision mode (intent and acceptance), not line-by-line micromanagement.
- Favor practical "build the thing quickly, then harden" loops for early project momentum.

Meijer-inspired:

- Treat formal structure and semantics as core, not decoration.
- Push correctness checks earlier in the loop (compile/semantic constraints during planning and execution, not just after edits).
- Prefer representations and tooling that preserve intent with minimal ambiguity.

Emanuel-inspired:

- Invest heavily in planning quality before long implementation runs.
- Keep operations explicit and reusable (prompts/plans/scripts) to create a compounding execution flywheel.
- Assume context is expensive and scarce: keep guidance concise, executable, and modular.

Steinberger-inspired:

- Ship at "inference speed" by parallelizing independent tasks and using focused toolchains.
- Bias toward local, scriptable workflows that reduce friction and round-trip latency.
- Evaluate full agent trajectories and operational behavior, not just superficial output quality.

Gauthier-inspired:

- Continuously benchmark against strong baselines.
- Treat retrieval/context strategy as a first-class system component.
- Use measurable leaderboards/metrics to guide iteration instead of intuition-only changes.

## Execution Doctrine

1. Operate by dependency graph, not by calendar

- Do not produce fake sprint timelines or "human-paced" milestone theater.
- Build and maintain a DAG of work packages.
- Prioritize highest-value unblocked nodes first.
- Recompute critical path after every major finding.

2. Maximize throughput with bounded risk

- Run independent tasks in parallel whenever tool/environment allows.
- Keep edits atomic and reversible.
- For risky edits, constrain blast radius to one subsystem before expansion.

3. Evidence before opinion

- Every architectural claim must be tied to either:
  - benchmark data,
  - reproducible experiment output, or
  - direct primary source evidence.
- Mark inference clearly when evidence is indirect.

4. Compiler-truth over text-appearance

- Prefer Roslyn semantic certainty over regex confidence.
- For C# changes touching symbols, references, or signatures, use semantic verification paths by default.

## Pit-Of-Success-First Doctrine

- Design command surfaces so the best workflow is the easiest workflow for agents.
- For high-traffic commands, ensure argument discovery is explicit via `describe-command` examples and guardrails.
- Maintain a short canonical startup path: `list-commands` -> `quickstart` -> targeted `describe-command`.
- Treat onboarding friction (argument confusion, bad first command choices, fallback churn) as a core quality metric, not docs polish.
- Keep release artifacts and skill packages aligned with this doctrine (include pit-of-success guidance near launchers).

## Operating Loop (Default)

For each work package:

1. Define goal + acceptance test
- State exact done criteria (build/test/analyzer/benchmark conditions).

2. Plan the operation graph
- List required prerequisites.
- Identify parallelizable branches.
- Define fastest safe sequence.

3. Execute at maximum safe speed
- Prefer deterministic scripts/tools over ad hoc manual repetition.
- Keep iteration cycle tight: edit -> validate -> record -> continue.

4. Validate with hard gates
- Compile success.
- Relevant tests pass.
- Analyzer policy respected.
- No unintended scope expansion.

5. Log learnings
- Capture what sped us up.
- Capture what caused churn/rework.
- Convert repeated lessons into agent rules.

## Planning and Specification Rules

- Spend substantial effort upfront on exact task specification when uncertainty is high.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DNAKode/RoslynSkills](https://github.com/DNAKode/RoslynSkills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
