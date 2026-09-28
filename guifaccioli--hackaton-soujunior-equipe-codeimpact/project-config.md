---
trigger: always_on
description: These rules apply to all work performed in this repository.
---

# Agent Guidelines

These rules apply to all work performed in this repository.

## Communication and feedback

- Be direct, objective, and technically honest.
- Do not praise ideas, decisions, implementations, or architecture merely to be agreeable.
- Do not soften technical criticism when something is clearly weak, incorrect, risky, unnecessary, overengineered, or poorly justified.
- When identifying a problem, explain:
  - what is wrong;
  - why it matters;
  - what concrete impact it may cause;
  - what simpler or safer alternative exists.
- Prefer useful technical feedback over validation, encouragement, or diplomatic wording.
- Distinguish clearly between:
  - confirmed issues;
  - assumptions;
  - trade-offs;
  - preferences;
  - future risks.
- Do not present a subjective preference as a technical requirement.
- If an existing solution is already sufficient, say so instead of proposing changes for the sake of improvement.
- If the requested approach is technically unsound, challenge it and explain why before implementing it.
- Avoid unnecessary verbosity. Lead with the conclusion, then provide the reasoning that materially affects the decision.

## YAGNI — You Aren’t Gonna Need It

- Do not implement functionality based only on hypothetical future requirements.
- Do not add abstractions, infrastructure, configuration, extensibility, indirection, or generalized architecture without a concrete current use case.
- Before adding complexity, verify:
  - Is this required by the current scope?
  - Is there a concrete use case for it now?
  - Is there an existing problem that this solves?
  - Does the benefit justify the additional maintenance cost?
- If the answer is no, prefer the simpler implementation.
- When rejecting unnecessary future-oriented complexity, explicitly state:

  `YAGNI — we don't need this yet.`
- Do not build systems around imagined scale, future teams, future tenants, future integrations, or future traffic unless those constraints are already part of the current requirements.
- Do not create extension points merely because something "might be useful later."
- Do not introduce generic abstractions when a concrete implementation is currently sufficient.
- Do not create configuration for values that do not currently need to vary.
- Do not add fallback mechanisms, compatibility layers, migration paths, or alternate providers unless there is a real current need for them.
- Prefer postponing complexity until the requirement actually exists.

## Prefer simple and boring solutions

- Prefer the simplest solution that correctly satisfies the current requirement.
- Favor predictable, conventional, easy-to-debug implementations over clever or highly abstract designs.
- Optimize primarily for:
  - correctness;
  - clarity;
  - maintainability;
  - ease of debugging;
  - low operational complexity.
- Avoid unnecessary architectural novelty.
- Do not introduce the following without a concrete justification tied to a current requirement:
  - microservices;
  - Redis;
  - queues;
  - event buses;
  - message brokers;
  - distributed workers;
  - API gateways;
  - premature caching;
  - generic repository layers;
  - unnecessary service layers;
  - excessive dependency injection;
  - plugin systems;
  - generalized adapters;
  - premature horizontal scaling;
  - complex deployment infrastructure;
  - unnecessary background processing.
- Prefer a direct function call over messaging when synchronous execution is sufficient.
- Prefer a single service or application over multiple deployable services when separation is not required.
- Prefer the existing database over adding another datastore unless the current database cannot reasonably solve the problem.
- Prefer existing project conventions over introducing new patterns without a clear benefit.
- Complexity must be a response to an observed requirement, not anticipation of a possible future one.

## Scope discipline

- Implement only what is necessary to satisfy the requested task.
- Do not refactor unrelated code while working on a focused change.
- Do not rename, move, reorganize, or modernize unrelated files unless required for the task.
- Do not expand the scope because an adjacent improvement appears convenient.
- Do not add dependencies unless they provide clear value for the current requirement and the same result cannot be achieved reasonably with existing tools.
- Do not create abstractions around code used only once unless there is a clear readability or correctness benefit.
- Preserve working behavior outside the requested scope.
- If you identify unrelated issues, report them separately instead of silently fixing them.

## Architecture decisions

- Start from the current requirements, not from a hypothetical ideal architecture.
- Prefer incremental architecture over speculative architecture.
- Before introducing a new architectural component, identify:
  - the current problem;
  - the failure mode of the simpler approach;
  - why the new component solves that problem;
  - the operational cost it introduces.
- Do not justify complexity using vague arguments such as:
  - "for scalability";
  - "for future growth";
  - "for best practices";
  - "for enterprise readiness";
  - "to make it more robust";  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GuiFaccioli/Hackaton-SouJunior-Equipe-CodeImpact](https://github.com/GuiFaccioli/Hackaton-SouJunior-Equipe-CodeImpact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
